# Custom AMI and Deploy the second Drupal instance

In this task you will update your AMI with the Drupal settings and deploy it in the second availability zone.

## Task 01 - Create AMI

### [Create AMI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/create-image.html)

Note : stop the instance before

|Key|Value for GUI Only|
|:--|:--|
|Name|AMI_DRUPAL_DEVOPSTEAM[XX]_LABO02_RDS|
|Description|Same as name value|

```bash
TODO redo this command
[INPUT]
aws ec2 create-image --instance-id i-0bc78af28a21cc344 --name AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS --description AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS \
--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS}]'


[OUTPUT]

{
    "ImageId": "ami-04bf78923175ffe07"
}


```

## Task 02 - Deploy Instances

* Restart Drupal Instance in Az1

* Deploy Drupal Instance based on AMI in Az2

|Key|Value for GUI Only|
|:--|:--|
|Name|EC2_PRIVATE_DRUPAL_DEVOPSTEAM[XX]_B|
|Description|Same as name value|

```bash
[INPUT]
# Restart Drupal Instance in AZ1 (A)
aws ec2 start-instances --instance-ids i-0caae283ae8f9517c

# Deploy Drupal Instance in AZ2
aws ec2 run-instances \
    --image-id <ami_id> \
    --instance-type <instance_type> \
    --subnet-id subnet-06579a70777df8833\
    --security-group-ids  vpc-03d46c285a2af77ba \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B},{Key=Description,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B}]'
[OUTPUT]
```

## Task 03 - Test the connectivity

### Update your ssh connection string to test

* add tunnels for ssh and http pointing on the B Instance

```bash
# TODO change ip address
ssh -i CLD_KEY_DMZ_SSH_CLD_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 -L 2223:10.0.10.7:22 

//updated string connection
```

## Check SQL Accesses

```sql
[INPUT]
//sql string connection from A
mysql -h <RDS_endpoint> -u <username> -p <password> -e "SELECT * FROM your_table;"
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com  -u admin -p -e "SELECT * FROM your_table;"

[OUTPUT]
```

```sql
[INPUT]
//sql string connection from B

[OUTPUT]
```

### Check HTTP Accesses

```bash
//connection string updated
curl http://localhost:8080

```

### Read and write test through the web app

* Login in both webapps (same login)

* Change the users' email address on a webapp... refresh the user's profile page on the second and validated that they are communicating with the same db (rds).

* Observations ?

```
//TODO
```

### Change the profile picture

* Observations ?

```
//TODO
```
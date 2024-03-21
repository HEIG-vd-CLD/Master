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
# Stop instance
aws ec2 stop-instances --instance-ids i-0caae283ae8f9517c
{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-0caae283ae8f9517c",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

# Create AMI
[INPUT] 
aws ec2 create-image --instance-id i-0caae283ae8f9517c --name AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS --description AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS \
--tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS}]' 

[OUTPUT]

{
    "ImageId": "ami-0900ddd620caee904"
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
# ip address 130 reserved, so 140
aws ec2 run-instances \
    --count 1 \
    --image-id ami-0900ddd620caee904 \
    --instance-type t3.micro \
    --key-name CLD_KEY_DRUPAL_DEVOPSTEAM10 \
    --private-ip-address 10.0.10.140 \
    --subnet-id subnet-06579a70777df8833\
    --security-group-ids sg-06b029dc68a7bf11b \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B},{Key=Description,Value=EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B}]'

[OUTPUT]
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0900ddd620caee904",
            "InstanceId": "i-00fcb741b3f8e348a",
            "InstanceType": "t3.micro",
            "KeyName": "CLD_KEY_DRUPAL_DEVOPSTEAM10",
            "LaunchTime": "2024-03-21T13:08:05+00:00",
            "Monitoring": {
                "State": "disabled"
:...skipping...
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0900ddd620caee904",
            "InstanceId": "i-00fcb741b3f8e348a",
            "InstanceType": "t3.micro",
            "KeyName": "CLD_KEY_DRUPAL_DEVOPSTEAM10",
            "LaunchTime": "2024-03-21T13:08:05+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "eu-west-3b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-10-140.eu-west-3.compute.internal",
            "PrivateIpAddress": "10.0.10.140",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-06579a70777df8833",
            "VpcId": "vpc-03d46c285a2af77ba",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "acc86ac1-bac4-45cb-a10f-15e7ab6f9f3b",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2024-03-21T13:08:05+00:00",
                        "AttachmentId": "eni-attach-0e0c95e2cc2f41c61",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "SG-PRIVATE-DRUPAL-DEVOPSTEAM10",
                            "GroupId": "sg-06b029dc68a7bf11b"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0a:70:34:94:81:57",
                    "NetworkInterfaceId": "eni-064539a2aa9524d8f",
                    "OwnerId": "709024702237",
                    "PrivateIpAddress": "10.0.10.140",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateIpAddress": "10.0.10.140"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-06579a70777df8833",
                    "VpcId": "vpc-03d46c285a2af77ba",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "SG-PRIVATE-DRUPAL-DEVOPSTEAM10",
                    "GroupId": "sg-06b029dc68a7bf11b"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B"
                },
                {
                    "Key": "Description",
                    "Value": "EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_B"
                }
            ],
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 2
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        }
    ],
    "OwnerId": "709024702237",
    "ReservationId": "r-062f5cf71307c8f2f"
}

```

## Task 03 - Test the connectivity

### Update your ssh connection string to test

* add tunnels for ssh and http pointing on the B Instance

```bash
# SUBNET A
ssh -i CLD_KEY_DMZ_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 \
-L 2223:10.0.10.7:22 -L 8080:10.0.10.7:8080
# SUBNET B
ssh -i CLD_KEY_DMZ_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 \
-L 2224:10.0.140.7:22 -L 8080:10.0.140.7:8080

ssh bitnami@localhost -p 2223 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem
ssh bitnami@localhost -p 2224 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem

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
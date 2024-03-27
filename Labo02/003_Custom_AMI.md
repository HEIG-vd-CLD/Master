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
# Get instance drupal A
# aws ec2 describe-instances --filters "Name=tag:Name,Values=EC2_PRIVATE_DRUPAL_DEVOPSTEAM10_A" --query "Reservations[*].Instances[*].InstanceId" --output text

aws ec2 stop-instances --instance-ids i-06aec672ad207f4c3

{
    "StoppingInstances": [
        {
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "InstanceId": "i-06aec672ad207f4c3",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}


# Create AMI
[INPUT] 
aws ec2 create-image \
--instance-id i-06aec672ad207f4c3 \
--name AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS \
--description AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS \
--tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS}]' 

[OUTPUT]

{
    "ImageId": "ami-0ef45705bb637c91f"
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
aws ec2 start-instances --instance-ids i-06aec672ad207f4c3
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-06aec672ad207f4c3",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
:...skipping...
{
    "StartingInstances": [
        {
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "InstanceId": "i-06aec672ad207f4c3",
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}
[OUTPUT]

# Deploy Drupal Instance in AZ2
# ip address 130 reserved, so set to 140
[INPUT]
aws ec2 run-instances \
    --count 1 \
    --image-id ami-0ef45705bb637c91f \
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
            "ImageId": "ami-0ef45705bb637c91f",
            "InstanceId": "i-010889f22f8bab126",
            "InstanceType": "t3.micro",
            "KeyName": "CLD_KEY_DRUPAL_DEVOPSTEAM10",
            "LaunchTime": "2024-03-23T13:32:45+00:00",
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
            "ClientToken": "ba846e55-15c2-4f94-a921-8478bce91555",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2024-03-23T13:32:45+00:00",
                        "AttachmentId": "eni-attach-0856a6afaff25145a",
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
                    "MacAddress": "0a:99:9d:34:0e:4b",
                    "NetworkInterfaceId": "eni-02af271f83ea2aee9",
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
    "ReservationId": "r-0d12fbc404958e570"
}

```

## Task 03 - Test the connectivity

### Update your ssh connection string to test

* add tunnels for ssh and http pointing on the B Instance

```bash

# SUBNET A
ssh -i CLD_KEY_DMZ_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 \
-L 2223:10.0.10.12:22 -L 8080:10.0.10.12:8080

# SUBNET B
ssh -i CLD_KEY_DMZ_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 \
-L 2224:10.0.10.140:22 -L 8081:10.0.10.140:8080

ssh bitnami@localhost -p 2223 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem
ssh bitnami@localhost -p 2224 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem

//updated string connection
```

## Check SQL Accesses

```sql
[INPUT]
//sql string connection from A
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u bn_drupal -p
// 2b9defd18a354804a1d4c4742c252fb39d808c12cfc2046ffc8f31432ae8a060
// Then in Mariadb
show databases;

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)
```

```sql
[INPUT]
//sql string connection from B
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u bn_drupal -p

// Then in Mariadb
show databases;

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)

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
The address is updated on both webapps. The two instances are communicating with the same database.
```

### Change the profile picture

* Observations ?

```
//TODO
  We are able to see the profile picture on only one of the webapps. For the other one, we can only see a broken image.
```
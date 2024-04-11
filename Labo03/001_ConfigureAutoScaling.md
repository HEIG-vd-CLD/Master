# Task 002 - Configure Auto Scaling

![Schema](./img/CLD_AWS_INFA.PNG)

* Follow the instructions in the tutorial [Getting started with Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/GettingStartedTutorial.html) to create a launch template.

* [CLI Documentation](https://docs.aws.amazon.com/cli/latest/reference/autoscaling/)

## Pre-requisites

* Networks (RTE-TABLE/SECURITY GROUP) set as at the end of the Labo2.
* 1 AMI of your Drupal instance
* 0 existing ec2 (even is in stopped state)
* 1 RDS Database instance - started
* 1 Elastic Load Balancer - started

## Create a new launch configuration. 

|Key|Value|
|:--|:--|
|Name|LT-DEVOPSTEAM[XX]|
|Version|v1.0.0|
|Tag|Name->same as template's name|
|AMI|Your Drupal AMI|
|Instance type|t3.micro (as usual)|
|Subnet|Your subnet A|
|Security groups|Your Drupal Security Group|
|IP Address assignation|Do not assign|
|Storage|Only 10 Go Storage (based on your AMI)|
|Advanced Details/EC2 Detailed Cloud Watch|enable|
|Purchase option/Request Spot instance|disable|

```
[INPUT]
//cli command
aws autoscaling create-launch-configuration \
    --launch-configuration-name LT-DEVOPSTEAM10 \
    --image-id $(aws ec2 describe-images \
    --query "Images[?Name=='AMI_DRUPAL_DEVOPSTEAM10_LABO02_RDS'].ImageId" --output text) \
    --instance-type t3.micro \
    --security-groups $(aws ec2 describe-security-groups \
    --query "SecurityGroups[?GroupName=='SG-PRIVATE-DRUPAL-DEVOPSTEAM10'].GroupId" --output text) \
    --key-name CLD_KEY_DRUPAL_DEVOPSTEAM10 \
    --associate-public-ip-address false \
    --block-device-mappings "[{\"DeviceName\":\"/dev/xvda\",\"Ebs\":{\"VolumeSize\":10}}]" \
    --instance-monitoring Enabled=true \
    --no-ebs-optimized
    --tags Key=Name,Value=LT-DEVOPSTEAM10

[OUTPUT]
TODO
```

## Create an auto scaling group

* Choose launch template or configuration

|Specifications|Key|Value|
|:--|:--|:--|
|Launch Configuration|Name|ASGRP_DEVOPSTEAM[XX]|
||Launch configuration|Your launch configuration|
|Instance launch option|VPC|Refer to infra schema|
||AZ and subnet|AZs and subnets a + b|
|Advanced options|Attach to an existing LB|Your ELB|
||Target group|Your target group|
|Health check|Load balancing health check|Turn on|
||health check grace period|10 seconds|
|Additional settings|Group metrics collection within Cloud Watch|Enable|
||Health check grace period|10 seconds|
|Group size and scaling option|Desired capacity|1|
||Min desired capacity|1|
||Max desired capacity|4|
||Policies|Target tracking scaling policy|
||Target tracking scaling policy Name|TTP_DEVOPSTEAM[XX]|
||Metric type|Average CPU utilization|
||Target value|50|
||Instance warmup|30 seconds|
||Instance maintenance policy|None|
||Instance scale-in protection|None|
||Notification|None|
|Add tag to instance|Name|AUTO_EC2_PRIVATE_DRUPAL_DEVOPSTEAM[XX]|

```
[INPUT]
//cli command
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name ASGRP_DEVOPSTEAM10 \
    --launch-configuration-name ASGRP_DEVOPSTEAM10 \
    --min-size 1 \
    --max-size 4 \
    --desired-capacity 1 \
    --vpc-zone-identifier $(aws ec2 describe-subnets \
    --filters "Name=tag:Name,Values=*DEVOPSTEAM10*" \
    --query "Subnets[*].SubnetId" \
    --output text \
     | paste -sd ",") \
    --health-check-type ELB \
    --health-check-grace-period 10 \
    --target-group-arns $(aws elbv2 describe-target-groups \
    --query "TargetGroups[?TargetGroupName=='TG-DEVOPSTEAM10'].TargetGroupArn" --output text) \
    --termination-policies "OldestLaunchConfiguration" \
    --tags Key=Name,Value=AUTO_EC2_PRIVATE_DRUPAL_DEVOPSTEAM10 \
    --new-instances-protected-from-scale-in \
    --metrics CollectionEnabled=True \
    --target-tracking-configuration file://target-tracking-configuration.json \

[OUTPUT]
```

* Result expected

The first instance is launched automatically.

Test ssh and web access.

```
[INPUT]
//ssh login
ssh -i CLD_KEY_DMZ_DEVOPSTEAM10.pem devopsteam10@15.188.43.46 -L 2223:10.0.10.12:22
ssh bitnami@localhost -p 2223 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem
[OUTPUT]
```

```
//screen shot, web access (login)
```
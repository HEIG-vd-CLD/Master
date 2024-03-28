### Deploy the elastic load balancer

In this task you will create a load balancer in AWS that will receive
the HTTP requests from clients and forward them to the Drupal
instances.

![Schema](./img/CLD_AWS_INFA.PNG)

## Task 01 Prerequisites for the ELB

* Create a dedicated security group

|Key|Value|
|:--|:--|
|Name|SG-DEVOPSTEAM[XX]-LB|
|Inbound Rules|Application Load Balancer|
|Outbound Rules|Refer to the infra schema|

```bash
[INPUT]
aws ec2 create-security-group \
    --group-name SG-DEVOPSTEAM10-LB \
    --description "Security group for the ELB" \
    --vpc-id vpc-03d46c285a2af77ba \
    --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=SG-DEVOPSTEAM10-LB}]'

[OUTPUT]
{
    "GroupId": "sg-0689100284e4dd5bc",
    "Tags": [
        {
            "Key": "Name",
            "Value": "SG-DEVOPSTEAM10-LB"
        }
    ]
}

[INPUT]
aws ec2 authorize-security-group-ingress \
    --group-id sg-0689100284e4dd5bc \
    --protocol tcp \
    --port 8080 \
    --cidr 10.0.0.0/28
    
[OUTPUT]
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00ffefdc0eb513716",
            "GroupId": "sg-0689100284e4dd5bc",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.0.0/28"
        }
    ]
}

```

* Create the Target Group

|Key|Value|
|:--|:--|
|Target type|Instances|
|Name|TG-DEVOPSTEAM[XX]|
|Protocol and port|Refer to the infra schema|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Protocol version|HTTP1|
|Health check protocol|HTTP|
|Health check path|/|
|Port|Traffic port|
|Healthy threshold|2 consecutive health check successes|
|Unhealthy threshold|2 consecutive health check failures|
|Timeout|5 seconds|
|Interval|10 seconds|
|Success codes|200|

```bash
[INPUT]
aws elbv2 create-target-group \
    --name TG-DEVOPSTEAM10 \
    --protocol HTTP \
    --port 8080 \
    --target-type instance \
    --health-check-protocol HTTP \
    --health-check-path / \
    --health-check-interval-seconds 10 \
    --health-check-timeout-seconds 5 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 2 \
    --vpc-id vpc-03d46c285a2af77ba \
    --protocol-version HTTP1 \
    --tags Key=Name,Value=TG-DEVOPSTEAM10

[OUTPUT]
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM10/49cb5841e91f6eeb",
            "TargetGroupName": "TG-DEVOPSTEAM10",
            "Protocol": "HTTP",
            "Port": 8080,
            "VpcId": "vpc-03d46c285a2af77ba",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "traffic-port",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 10,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 2,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/",
            "Matcher": {
                "HttpCode": "200"
            },
            "TargetType": "instance",
            "ProtocolVersion": "HTTP1",
            "IpAddressType": "ipv4"
        }
    ]
}

[INPUT]
# register instances A and B to the target group
aws elbv2 register-targets \
    --target-group-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM10/49cb5841e91f6eeb \
    --targets Id=i-06aec672ad207f4c3 Id=i-010889f22f8bab126

[OUTPUT]
None
```


## Task 02 Deploy the Load Balancer

[Source](https://aws.amazon.com/elasticloadbalancing/)

* Create the Load Balancer

|Key|Value|
|:--|:--|
|Type|Application Load Balancer|
|Name|ELB-DEVOPSTEAM10|
|Scheme|Internal|
|Ip Address type|IPv4|
|VPC|Refer to the infra schema|
|Security group|Refer to the infra schema|
|Listeners Protocol and port|Refer to the infra schema|
|Target group|Your own target group created in task 01|

Provide the following answers (leave any
field not mentioned at its default value):

```bash
[INPUT]

aws elbv2 create-load-balancer \
    --name ELB-DEVOPSTEAM10 \
    --type application \
    --scheme internal \
    --ip-address-type ipv4 \
    --subnets subnet-02291f4084cacd8c9 subnet-06579a70777df8833 \
    --security-groups sg-0689100284e4dd5bc \
    --tags Key=Name,Value=ELB-DEVOPSTEAM10


[OUTPUT]
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM10/7c088baef9402321",
            "DNSName": "internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z3Q77PNBQS71R4",
            "CreatedTime": "2024-03-23T14:27:59.600000+00:00",
            "LoadBalancerName": "ELB-DEVOPSTEAM10",
            "Scheme": "internal",
            "VpcId": "vpc-03d46c285a2af77ba",
            "State": {
                "Code": "provisioning"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "eu-west-3a",
                    "SubnetId": "subnet-02291f4084cacd8c9",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "eu-west-3b",
                    "SubnetId": "subnet-06579a70777df8833",
                    "LoadBalancerAddresses": []
                }
            ],
            "SecurityGroups": [
                "sg-0689100284e4dd5bc"
            ],
            "IpAddressType": "ipv4"
        }
    ]
}

```

# Create the listener

```bash
[INPUT]
aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM10/7c088baef9402321 \
    --protocol HTTP \
    --port 8080 \
    --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM10/49cb5841e91f6eeb

[OUTPUT]
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:listener/app/ELB-DEVOPSTEAM10/7c088baef9402321/54fc535210bf9a6a",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:loadbalancer/app/ELB-DEVOPSTEAM10/7c088baef9402321",
            "Port": 8080,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM10/49cb5841e91f6eeb",
                    "ForwardConfig": {
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-3:709024702237:targetgroup/TG-DEVOPSTEAM10/49cb5841e91f6eeb",
                                "Weight": 1
                            }
                        ],
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        }
                    }
                }
            ]
        }
    ]
}

```

* Get the ELB FQDN (DNS NAME - A Record)

```bash
[INPUT]
aws elbv2 describe-load-balancers --names ELB-DEVOPSTEAM10 --query "LoadBalancers[0].DNSName"


[OUTPUT]

"internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com"

```

* Get the ELB deployment status

Note : In the EC2 console select the Target Group. In the
       lower half of the panel, click on the **Targets** tab. Watch the
       status of the instance go from **unused** to **initial**.

* Ask the DMZ administrator to register your ELB with the reverse proxy via the private teams channel

* Update your string connection to test your ELB and test it

```bash
// add ips 8080 port for DMZ security group
aws ec2 authorize-security-group-ingress --group-id sg-06b029dc68a7bf11b --protocol tcp --port 8080 --cidr 10.0.10.0/28
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-04d157247d5ff4d96",
            "GroupId": "sg-06b029dc68a7bf11b",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.10.0/28"
        }
    ]
}

aws ec2 authorize-security-group-ingress --group-id sg-06b029dc68a7bf11b --protocol tcp --port 8080 --cidr 10.0.10.128/28
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0676cab0f691bb3f8",
            "GroupId": "sg-06b029dc68a7bf11b",
            "GroupOwnerId": "709024702237",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "10.0.10.128/28"
        }
    ]
}

//connection string updated

ssh devopsteam10@15.188.43.46 -i CLD_KEY_DMZ_DEVOPSTEAM10.pem \
-L 2226:internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com:8080 

```

* Test your application through your ssh tunneling

```bash
[INPUT]
// in host
curl localhost:2226

[OUTPUT]

<!DOCTYPE html>
<html lang="en" dir="ltr" style="--color--primary-hue:202;--color--primary-saturation:79%;--color--primary-lightness:50">
  <head>
    <meta charset="utf-8" />
<meta name="Generator" content="Drupal 10 (https://www.drupal.org)" />
<meta name="MobileOptimized" content="width" />
<meta name="HandheldFriendly" content="true" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<link rel="icon" href="/core/themes/olivero/favicon.ico" type="image/vnd.microsoft.icon" />
<link rel="alternate" type="application/rss+xml" title="" href="http://localhost:8080/rss.xml" />
(...)

```

#### Questions - Analysis

* On your local machine resolve the DNS name of the load balancer into
  an IP address using the `nslookup` command (works on Linux, macOS and Windows). Write
  the DNS name and the resolved IP Address(es) into the report.

```
nslookup internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com
[OUTPUT]
(...)
Name:   internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com
Address: 10.0.10.132
Name:   internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com
Address: 10.0.10.14
```

* From your Drupal instance, identify the ip from which requests are sent by the Load Balancer.

Help : execute `tcpdump port 8080`

```
// needs ssh tunneling
ssh devopsteam10@15.188.43.46 -i CLD_KEY_DMZ_DEVOPSTEAM10.pem \
 -L 2225:10.0.10.12:22 \
-L 80:internal-ELB-DEVOPSTEAM10-394516614.eu-west-3.elb.amazonaws.com:8080

ssh bitnami@localhost -p 2225 -i CLD_KEY_DRUPAL_DEVOPSTEAM10.pem
sudo tcpdump port 8080
```

* In the Apache access log identify the health check accesses from the
  load balancer and copy some samples into the report.

```
// in bitnami
cat /opt/bitnami/apache/logs/access_log

10.0.10.14 - - [27/Mar/2024:19:30:42 +0000] "GET / HTTP/1.1" 200 5146
10.0.10.132 - - [27/Mar/2024:19:30:49 +0000] "GET / HTTP/1.1" 200 5146
10.0.10.14 - - [27/Mar/2024:19:30:52 +0000] "GET / HTTP/1.1" 200 5146
10.0.10.132 - - [27/Mar/2024:19:30:59 +0000] "GET / HTTP/1.1" 200 5146
10.0.10.14 - - [27/Mar/2024:19:31:02 +0000] "GET / HTTP/1.1" 200 5146
10.0.10.132 - - [27/Mar/2024:19:31:09 +0000] "GET / HTTP/1.1" 200 5146
(...)
```

### Deploy the elastic load balancer

In this task you will create a load balancer in AWS that will receive
the HTTP requests from clients and forward them to the Drupal
instances.

[//]: # (TODO note: nothing executed yet)
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

[INPUT]
aws ec2 authorize-security-group-ingress \
    --group-id <created_sg> \
    --protocol tcp \
    --port 8080 \
    --cidr 10.0.10.0/28

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
    --target-group-attributes Key=deregistration_delay.timeout_seconds,Value=300 \
    --protocol-version HTTP1 \
    --tags Key=Name,Value=TG-DEVOPSTEAM10


[OUTPUT]

[INPUT]
# register instances A and B to the target group
aws elbv2 register-targets \
    --target-group-arn <target_group_arn> \
    --targets Id=i-0caae283ae8f9517c Id=i-0caae283ae8f9517d

[OUTPUT]

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
    --security-groups <security_group_ids_LB> \
    --tags Key=Name,Value=ELB-DEVOPSTEAM10


[OUTPUT]

```

# Create the listener

```bash
[INPUT]
aws elbv2 create-listener \
    --load-balancer-arn <load_balancer_arn> \
    --protocol HTTP \
    --port 8080 \
    --default-actions Type=forward,TargetGroupArn=<target_group_arn>

[OUTPUT]

```

* Get the ELB FQDN (DNS NAME - A Record)

```bash
[INPUT]
aws elbv2 describe-load-balancers --names ELB-DEVOPSTEAM10 --query "LoadBalancers[0].DNSName"


[OUTPUT]

```

* Get the ELB deployment status

Note : In the EC2 console select the Target Group. In the
       lower half of the panel, click on the **Targets** tab. Watch the
       status of the instance go from **unused** to **initial**.

* Ask the DMZ administrator to register your ELB with the reverse proxy via the private teams channel

* Update your string connection to test your ELB and test it

```bash
//connection string updated

ssh devopsteam10@15.188.43.46 -i CLD_KEY_DMZ_DEVOPSTEAM10.pem \
-L 2225:10.0.10.7:22 \
-L 8080:<load_balancer_dns_name>:8080 

```

* Test your application through your ssh tunneling

```bash
[INPUT]
curl localhost:8080

[OUTPUT]

```

#### Questions - Analysis

* On your local machine resolve the DNS name of the load balancer into
  an IP address using the `nslookup` command (works on Linux, macOS and Windows). Write
  the DNS name and the resolved IP Address(es) into the report.

```
//TODO
nslookup <load_balancer_dns_name>
```

* From your Drupal instance, identify the ip from which requests are sent by the Load Balancer.

Help : execute `tcpdump port 8080`

```
//TODO
sudo tcpdump port 8080


```

* In the Apache access log identify the health check accesses from the
  load balancer and copy some samples into the report.

```
//TODO

cat /opt/bitnami/apache/logs/access_log

```

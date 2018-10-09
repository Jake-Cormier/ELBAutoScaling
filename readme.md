# ELB and Auto Scaling

![Diagram of Design](https://i.imgur.com/PAImU4V.png)

## Breakdown

VPC with a front facing classic ELB, distributing traffic to an auto scaling group. The auto scaling group will be spread across two AZs with a minimum of 2 EC2 instances and policies for scaling up and down.

## Auto Scaling

The launch configuration will be a standard Amazon Linux AMI with a t2.mirco (just for free tier) instance. To set the instance up to par on start a simple bash script will be installed, with an update and apache server installation. The apache web server is for the testing of the ELB and auto scaling group

    #!/bin/bash
    yum update -y
    yum install -y httpd
    service httpd start

The public instances will also be assigned public IP addresses. The security group will allow both SSH and HTTP rules, HTTP with the standard 0.0.0.0/0 open to the world source.  The auto scaling group will also start with 2 instances and max out at 4 instances.

Scale up `Whenever: Average of CPU Utilization is: >= 80% for at least 5 minutes provision another instance.`

Scale down `Whenever: Average of CPU Utilization is: <= 40% for at least 5 minutes remove a provisioned instance.`

This will allow for optimal scalability and elasticity

## ELB

The internet facing classic ELB will be configured with the listener configuration as HTTP over port 80 with both public subnets in the VPC selected.  The ELB will also have the default configurations of a new security group, TCP rule over port 80 with the source open to the world. Since this is practice with ELB and auto scaling a page will not be set up, so the health check's ping protocol will need to be configured over TCP and not HTTP.


## Testing

Using the ELB DNS name found in the ELB description you can simply copy and paste the link in a new tab. This will launch the default apache test page, assuring the ELB is pointing the traffic to one of the EC2 instances in the auto scaling group. Since there is no data to change CPU levels of the instances, the best thing to do is to manually terminate an instance to see if the auto scaling group provisions a new instance after the length of its health checks.

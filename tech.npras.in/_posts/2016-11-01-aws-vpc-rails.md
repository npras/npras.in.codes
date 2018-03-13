---
layout: post
title: "AWS VPC Architecture for a highly-available, scalable Rails Application"
categories: aws
excerpt: Case study of how I built a Highly-Available Infrastructure for a Client's Rails App within the AWS Ecosystem
---

Recently I worked on a client project where the requirements were this:

* In **Phase 1**, build a Rails API Application that would eventually cater to millions of requests in a day
* In **Phase 2**, build a "stack" for this application in the AWS Ecosystem using services like EC2, RDS, Opsworks, ELB, VPC, Subnets, Security Groups etc

This post details about the second phase where I had a working rails application in hand, that I wanted to deploy it in a secure infrastructure within AWS.

The client had a few **requirements** as to how the security aspect of the infrastructure would be:

* The Rails Application should be served by Application Servers that are not publicly accessible. That means no Elastic IP or public IPs attached to them. It also means the app servers would reside in a subnet that's not accessible directly from the internet - **a private subnet**.
* The requests to the application be served to the app servers from AWS's **Elastic LoadBalancer**. No requests should (or can) hit the app servers directly.
* Though the app servers shouldn't be reachable from the internet, the app's themselves should be able to talk to the internet and get response. Case in question: The rails app talks to some 3rd party services via their API. To enable this, the private subnets should have all of their outgoing traffic routed to a **NAT Gateway** present in the public subnet.
* To be able to SSH into the app server instances present in the private subnets, there should be a **"Bastion" server** instance running in a public subnet. The Bastion server will have a public or Elastic IP attached to it, through which we can SSH into it from the internet. And from within the bastion server, you may SSH into any of the app servers present in the private subnet.
* **High Availability** for the App servers and **Automatic failover** for the RDS database instance for the app.
* Strict Security Groups to allow only the necessary traffic that's coming and going out of the instances.
* The Application setup and deployment should be configured using Chef using AWS Opsworks service.

With these requirements in hand, I set out to design and develop the following **infrastructure**.

* 1 VPC.
* 4 subnets - 2 public and 2 private. The reason for 2 of each is this: Each of a group (private or public) will reside in a different Availability Zone compared to the other. This ensures High-Availability. Hence the 2 private subnets. The ELB for the app server that resides in the public subnets should also be made highly available. Hence the 2 public subnets.
* 2 Route Tables: 1 is associated to 2 public subnets, and another to the 2 private subnets.
* 1 Internet Gateway attached to the public subnets. This allows the instances in the public subnets to talk to the Internet.
* 1 NAT Gateway attached to the public subnets. This allows the instances in the private subnets to talk to the Internet.
* An Elastic IP attached to the NAT Gateway. This allows the rails app to talk to 3rd party API services from within the app.
* 2 Route Table entries (aside from the 'local' default entries): 1 entry is for the private route table allowing the outbound traffic to pass through the NAT. The other entry is for the private route table allowing the outbound traffic to pass through the Internet Gateway.
* 1 ELB that is configured to provide High-Availability from the 2 public subnets.
* The ELB has SSL certificate attached to it, and configured to forward traffic from ELB's port 443 (https) to the App Servers' port 80 (http).
* 1 RDS DB Instance that belongs to a **DBSubnetGroup** that is configured to *failover* from the 2 private subnets (that has the app server instances).
* AWS doesn't provide high-availability for RDS like ELB. Instead, it does these:
 If the primary DB instance of a Multi-AZ deployment fails, Amazon RDS can promote the corresponding standby and subsequently create a new standby using an IP address of the subnet in one of the other Availability Zones.
* 1 Bastion Server in one of the public subnets.
* Relevant Security Groups.

This architecture closely resembles the ['Scenario 2'](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenario2.html) example seen in AWS Documentation, with slight changes.

The whole setup was later automated via AWS Cloudformation as a parameterized json template. With just that template and the necessary params, the client was able to spin the app and its infrastructure in various AWS regions around the world.

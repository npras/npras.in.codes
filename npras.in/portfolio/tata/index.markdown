---
layout: page
---

([**portfolio**](/portfolio/) > **tata**)


## Sponsor Quota Manager for Sponsored Data Exchange


#### __When did you work on this project?__
8 months from May to December 2016.


#### __What is this project about?__
The project was conceived based on a [patent](https://www.google.com/patents/US20150242903) developed and owned by the client. They came with a design plan and a list of deliverables scheduled to be delivered over the next 3 months. Tata initially preferred this be done using Python and Django, as the lead Architects at the client-side were only familiar with that. But we convinced them to do it in Ruby and Rails.

The project is to allow Tata to venture into the Sponsored Data Exchange business in their telecom vertical in USA. It is a service and system for providing sponsored data, wherein the data usage is not charged to a mobile subscriber but, rather, paid for by a sponsor.


#### __Who was the client?__
[__Tata Communications (America) Inc__](http://www.tatacommunications.com/).


#### __What did _you_ do exactly?__
In a span of 4 months, I implemented 3 projects (2 Rails, 1 RubyScript), designed a cloud-scale infrastructure, and then deployed them to the cloud using modern devops tools.

I lead a team of 3, did over 80% of the design and development. I was responsible for client communication for requirement gathering and status reporting.

In the first phase, I built the 2 Rails apps that exposed many APIs. I used Rails 5. Based on the client’s high-level model requirements, I designed the low-level database and entities with relationships. I had to take care of all query performances too. I wrote test cases for all APIs and made sure the test coverage is over 90%. I also used rubo-cop gem to enforce a coding standard among all members of the team.

For billing requirements, the client wanted to use a NoSql DB too, apart from the traditional Mysql RDBMS. We chose AWS’s DynamoDB, and implemented the right partition and sort keys that would prove crucial to building the RubyScript app later on.

In the second phase, I designed and deployed the infrastructure required to scale this project in the future, in AWS.

* I used Opsworks to setup these 3 app in 3 different stacks.
* I wrote custom Chef recipes, when Opsworks’ in-built Chef Recipes fell short.
* I defined a secure architecture where these apps would run within AWS: I created
  -  VPC,
  - private and public subnets,
  - route tables and routes to define traffic among them,
  - NAT Gateway and Internet Gateway to enable the network to talk to internet,
  - Security Groups to control access to ports,
  - Elastic Loadbalancers to distribute traffic among multiple instances running the same application,
  - Cloudwatch metrics and alarms to watch important KPIs
  - Setup Loadbased Autoscaling instances that would come to life when the load on app -instances go beyond a certain threshold
  - Defined IAM Users and Roles to provide/restrict access to these resources


#### __Process/Technologies Used__
* Rails 5, Ruby 2.3.1, MySql 5.6
* [AASM Statemachine](https://github.com/aasm/aasm) for implementing complex state transitions
* MacOS for development, Ubuntu for Server
* Chef 11.10 in AWS Opsworks
* AWS Services - RDS, EC2, Elastic Loadbalancers, Elastic IPs, Opsworks, IAM, Cloudwatch alarms etc
* Agile approach with weekly scrum meetings with the client, and twice-a-week meeting internally.
* Since the client was in USA, I worked remotely in the project.


#### __How did your work impact the company?__
The client was able to take the final version of the deliverables and showcase it to a few enterprise customers (sponsors) who have expressed interest in this product. This is set out to become a multi-million dollar project in 2017, deployed around various parts of the world.


#### __Was the client happy?__
The client was extremely satisfied with my work and appreciated the delivery on time. In many places in the initial design spec, we faced issues. The alternative technical solutions I proposed went well with the client who agreed to incorporate these changes in their design doc.


#### __More info?__
I've written some blog posts about the things I learned from the project. Here are some:

* [Deviseless, Headerless Authentication in Rails API](http://tech.npras.in/deviseless-headerless-authentication-in-rails-api/)
* [DynamoDB: Fetching Latest Items By Time: Part 1: Choosing the right Primary Keys](http://tech.npras.in/dynamodb-timebased-items-query-1/)
* [DynamoDB: Fetching Latest Items By Time: Part 2: Scan and Query APIs coupled with some Logic](http://tech.npras.in/dynamodb-timebased-items-query-2/)
* [AWS Cloudformation: Template Generation Strategy](http://tech.npras.in/aws-cloudformation-template-generation/)
* [AWS VPC Architecture for a highly-available, scalable Rails Application](http://tech.npras.in/aws-vpc-rails/)
* [AWS API Authentication With InstanceProfile in Ruby](http://tech.npras.in/aws-api-authentication-with-instanceprofile/)
* [Setting up LoadBased AutoScaling Instance in Opsworks Using CloudFormation](http://tech.npras.in/loadbased-autoscaling-opsworks-cloudformation/)

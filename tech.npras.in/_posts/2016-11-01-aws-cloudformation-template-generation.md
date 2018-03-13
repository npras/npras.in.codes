---
layout: post
title: "AWS Cloudformation: Template Generation Strategy"
categories: aws
excerpt: "Learn how to incrementally build the template programmatically and effectively"
---

In one of the recent client projects, I developed 2 Rails apps and a Ruby Script that will be run via cron. All of these were to be created within the AWS Infrastructure using services like EC2, RDS, Opsworks, VPC, Subnet, ELBs, Security Groups, Elastic IPs etc.

However the final deliverable is the Cloudformation json template file which has the whole application and architecture embedded as json objects. This way, the client can then take the template and deploy it in different AWS Regions around the world, and have the project deployed quickly and consistently.

While Cloudformation is a nice strategy and a step forward in the world of DevOps automation, the [designer tool](https://aws.amazon.com/blogs/aws/new-aws-cloudformation-designer-support-for-more-services/) that AWS provides to create it is horrendous.

The main disadvantages of using the Designer tool:

* You are just creating a json file. You can do that in your favorite text editor. But AWS wants you to build your client's complex infrastructure (see image below) in this tool. You are leaving the comfort zone of your editor.
* Beyond a point when your infrastructure has to have many components, putting all the boxes and linking them up in one screen doesn't give you the big picture view. You'll have to zoom in and out, and drag here and there to just move around. Plenty of ways to make mistakes.

So I propose you build the json file incrementally, using any programming language of your choice.

Here's the simplest version in Ruby:

```rb
def process!
  h = {}

  h['AWSTemplateFormatVersion'] = '2010-09-09'
  h['Description'] = "My App's Cloudformation Template"
  h['Metadata'] = cf_metadata
  h['Parameters'] = cf_parameters
  h['Mappings'] = cf_mappings
  h['Conditions'] = cf_conditions
  h['Resources'] = cf_resources
  h['Outputs'] = cf_outputs

  create_template(h)
end


def create_template(h)
  File.open(FILE_PATH_JSON_TEMPLATE, "wb") do |f|
    f.write JSON.pretty_generate(h)
  end
  File.open(FILE_PATH_YAML_TEMPLATE, "wb") do |f|
    f.write h.to_yaml
  end
end
```

Where the literal tokens beginning with "cf\_" are all methods that return the respective hashes. Using this approach, you can build the hash objects for any level.

For instance, the `cf_parameters` method returns the Parameters data:

```rb
def cf_parameters
  h = {}

  h["ParamVPCCidrBlock"] = {
    "Type": "String",
    "Default": "10.0.0.0/16",
    "Description": "CIDR block for VPC",
  }

  h
end
```

And when creating multiple resources of the same resource type, you can write a single method taking arguments, depending on which the output hash object will change. For instance, this method can create different subnets based on the passed params:

```rb
def cloudformation_subnet(name:, vpc:, public_ip: 'true', param_cidr:, param_az:)
  {
    "Type": "AWS::EC2::Subnet",
    "DependsOn": vpc,
    "Properties": {
      "AvailabilityZone": { "Ref": param_az },
      "CidrBlock": { "Ref": param_cidr },
      "MapPublicIpOnLaunch": public_ip,
      "Tags": [{ "Key": "Name", "Value": name }],
      "VpcId": { "Ref": vpc }
    }
  }
end
```

## Outro
This image was created from the AWS Designer tool. But not by creating it manually. I just copy-pasted the JSON I created using the above method, into the designer tool. After refreshing it, the designer automatically generated the diagram based on the json content.

Imagine how difficult and error-prone it would have been if I had to build this infra manually using the designer!

![AWS Cloudformation Designer Architecture]({{site.url}}/assets/images/aws-cloudformation-designer-architecture.png)

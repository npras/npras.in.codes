---
layout: post
title: "AWS API Authentication With InstanceProfile in Ruby"
categories: aws
excerpt: "Learn how to make API calls using the Ruby SDK, using InstanceProfile, without using ACCESS_KEY_ID and SECRET_ACCESS_KEY"
---

The official [documentation for AWS Ruby SDK](http://docs.aws.amazon.com/sdkforruby/api/index.html) lists out ways to configure credentials.

The most common way would be to use your IAM User's ACCESS_KEY_ID and SECRET_ACCESS_KEY. In ruby, you can configure the `Aws.config` hash to take these as credentials like so:

```rb
# config/initializers/aws.rb

require 'aws-sdk'

credentials = Aws::Credentials.new(ENV['AWS_ACCESS_KEY_ID'],
                  ENV['AWS_SECRET_ACCESS_KEY'])

Aws.config.update({
  region: ENV['AWS_REGION'],
  credentials: credentials
})

```

But AWS actively discourages this approach. They suggest an alternate way to manage the credentials.

By assigning an IAM Role to an EC2 Instance while launching it, and adding the required policies (permissions) to this IAM role. This would allow all applications running on that EC2 instance make API calls to the authenticated services.

To implement it in the above AWS Initializer file, you'll have to change the credentials object like so:

```rb
# config/initializers/aws.rb

require 'aws-sdk'

credentials = Aws::InstanceProfileCredentials.new

Aws.config.update({
  region: ENV['AWS_REGION'],
  credentials: credentials
})
```

Now, you'll have to make sure the EC2 this app is running on, is attached to a role that has the necessary policy attached to it.

## Cloudformation Setup

Suppose the above app wants to make DynamoDB API calls, then we can create the necessary IAM Role and Instance Profile and associate them in the 'Instance Profile' setting of your [EC2 Instance json](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html#cfn-ec2-instance-iaminstanceprofile) or the [Opsworks Stack json](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-opsworks-stack.html#cfn-opsworks-stack-defaultinstanceprof).

Here I'll show the example of doing it in the Opsworks Stack json.

First, here's the cloudformation json, you'd need for your IAM Role:

```json
{
  "Type": "AWS::IAM::Role",
  "DependsOn": "vpc",
  "Properties": {
    "AssumeRolePolicyDocument": {
      "Version": "2008-10-17",
      "Statement": [
        {
          "Sid": "",
          "Effect": "Allow",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    },
    "Path": "/",
    "Policies": [
      {
        "PolicyName": "DynamoDBAccess",
        "PolicyDocument": {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Action": [
                "dynamodb:*",
                "cloudwatch:DeleteAlarms",
                "cloudwatch:DescribeAlarmHistory",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:DescribeAlarmsForMetric",
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:ListMetrics",
                "cloudwatch:PutMetricAlarm",
                "datapipeline:ActivatePipeline",
                "datapipeline:CreatePipeline",
                "datapipeline:DeletePipeline",
                "datapipeline:DescribeObjects",
                "datapipeline:DescribePipelines",
                "datapipeline:GetPipelineDefinition",
                "datapipeline:ListPipelines",
                "datapipeline:PutPipelineDefinition",
                "datapipeline:QueryObjects",
                "iam:ListRoles",
                "sns:CreateTopic",
                "sns:DeleteTopic",
                "sns:ListSubscriptions",
                "sns:ListSubscriptionsByTopic",
                "sns:ListTopics",
                "sns:Subscribe",
                "sns:Unsubscribe",
                "sns:SetTopicAttributes",
                "lambda:CreateFunction",
                "lambda:ListFunctions",
                "lambda:ListEventSourceMappings",
                "lambda:CreateEventSourceMapping",
                "lambda:DeleteEventSourceMapping",
                "lambda:GetFunctionConfiguration",
                "lambda:DeleteFunction"
              ],
              "Effect": "Allow",
              "Resource": "*"
            }
          ]

        }
      },
    ],
    "RoleName": "aws-opsworks-ec2-my-role"
  }
}
```

Then, here's the cloudformation json you'd need for associating the above IAM Role to an IAM InstanceProfile:

```json
{
  "Type": "AWS::IAM::InstanceProfile",
  "DependsOn": ["vpc", "role"],
  "Properties": {
    "Path": "/",
    "Roles": [
      { "Ref": "role" },
    ]
  }
}  
```

Where the role referenced is the role we created above.

Now you can use this InstanceProfile in your Opsworks stack's `DefaultInstanceProfileArn` property.

Then, all instances launched from this opsworks stack, will now have this DynamoDB access for the applications running in them.

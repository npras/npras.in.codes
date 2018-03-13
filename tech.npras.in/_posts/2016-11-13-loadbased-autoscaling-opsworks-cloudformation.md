---
layout: post
title: "Setting up LoadBased AutoScaling Instance in Opsworks Using CloudFormation"
categories: "aws"
excerpt: "Learn how to 'Scale in Cloud' using Loadbased Instances in AWS Opsworks"
---

Apart from being able to create normal "24/7" type instances in your AWS Opsworks Stack, you can also configure a Load or Time based autoscaling instances to suit your needs.

A LoadBased instance will come to life when the average CPU/Memory % of your normal instances go beyond a certain threshold. The load to your application (typically from ELB) will then be shared by 1 or more of these autoscaling instances.

Here are the steps required to configure it manually:

* In your chosen Opsworks stack, in the "Instances" section, choose "Load-based" subsection. There, 'Edit Configuration' to define your threshold.
* Click the "+ Instance" or Add Instance button in the "Load-based" subsection page. Define your Instance settings, and click "Add Instance".

This instance will not boot and come to online right away. Only when the average load among the normal 24/7 servers go up, this instance will boot and come online to save the day.

## Cloudformation Setup
Here's what you have to do to add autoscaling instance in your cloudformation template.

1) Define the "LoadbasedAutoScaling" property in your "AWS::OpsWorks::Layer" like so:

```json
"LoadBasedAutoScaling": {
  "Enable": "true",
  "DownScaling": {
    "InstanceCount": 1,
    "ThresholdsWaitTime": 5,
    "IgnoreMetricsTime": 10,
    "CpuThreshold": 10.0,
    "MemoryThreshold": 15.0
  },
  "UpScaling": {
    "InstanceCount": 1,
    "ThresholdsWaitTime": 5,
    "IgnoreMetricsTime": 15,
    "CpuThreshold": 80.0,
    "MemoryThreshold": 85.0
  }
}

```

2) While defining the Opsworks Instance, add the "AutoScalingType" property, and set it to "load":

```json
"OpsInstance": {
  "Type": "AWS::OpsWorks::Instance",
  "Properties": {
    "Architecture": "x86_64",
    "InstallUpdatesOnBoot": "true",
    "InstanceType": "t2.micro",
    "Os": "Ubuntu 14.04 LTS",
    "RootDeviceType": "ebs",
    "SshKeyName": "somekey",
    "StackId": {
      "Ref": "OpsStack"
    },
    "SubnetId": {
      "Ref": "Subnet1"
    },
    "LayerIds": [
      {
        "Ref": "OpsLayer"
      }
    ],
    "AutoScalingType": "load"
  }
}
```

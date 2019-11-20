---
title: "Lab 04: Mapping"
date: 2019-11-01T13:36:34Z
weight: 400
---

## Introduction

This lab will cover Mappings, which allow you to lookup values from a set of predefined keys.

A common use for Mappings is to configure a template according to different environments, such as dev, test and production.
 Mapping allows you to have one template, rather than 3 similar templates. 
 
 Keys and their corresponding values are predefined in the `Mappings` section. The value of these keys are referenced in other parts of your CloudFormation template.

### Key Components

#### Mapping Section

The [`Mappings`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html) section is a top level section of a CloudFormation template. It is used to define maps, their keys and values.


A Mappings Section contains one or more maps. Each map contains at least one key. \
Each key is another map! It contains one or more keys that map to values.
Each key contains one or more Name - Value pairs. Each top level key  contains one or more second level keys.


Here is a simplified example of a Mappings section. It contains one Map, `AnExampleMapping`. \
`AnExampleMapping` contains three top level keys, `Key01`, `Key02` and `Key03`. \
Each top level key contains one or more `Name`:`Value` pairs.

```yaml
Mappings: 
  AnExampleMapping: 
    Key01: 
      Name: Value01
      AnotherName: Value02
    Key02: 
      Name: Value02
    Key03: 
      Name: Value03
```


## Implementing a simple map

Below is a simple CloudFormation template. It uses simple mappings section to configure the EC2 instance type according to a parameter, `EnvironmentType`.

```yaml
Parameters:
  EnvironmentType: 
    Description: "Specify the Environment type of the stack"
    Type: String
    Default: "Dev"
    AllowedValues:
      - "Dev"
      - "Test"

    
  AmiID:
    Type: AWS::EC2::Image::Id
    Description: 'Amazon Machine Image ID'

Mappings:
  EnvironmentToInstanceType: # Map Name
    Dev: # Top level key
      InstanceType: "t3.micro" # Second level key
    Test:
      InstanceType: "t3.nano"

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties: 
      ImageId: !Ref AmiID

      # Use the intrinsic function FindInMap to lookup the 
      # InstanceType value from the EnvironmentToInstanceTypeMap.
      # It references the EnvironmentType parameter provided to the template
      InstanceType: !FindInMap
        - EnvironmentToInstanceType # Map Name
        - !Ref EnvironmentType # Top Level Key
        - "InstanceType" # Second Level Key
```

The mapping section defines one map, `EnvironmentToInstanceType`.
The map contains two top level keys, one for each environment.
Each top level key contains a single `InstanceType` second level key.
```yaml
Mappings:
  EnvironmentToInstanceType: # Map Name
    Dev: # Top level key
      InstanceType: "t3.micro" # Second level key
    Test:
      InstanceType: "t3.nano"
```

The `InstanceType` property defines the type of EC2 instance. The intrinsic function `Fn::FindInMap` is used to lookup the value in the `EnvironmentToInstanceType` map.
The parameter `EnvironmentType` is passed as the top level key using the intrinsic function `Fn::Ref`.

```yaml
Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties: 
      ImageId: !Ref AmiID

      # Use the intrinsic function FindInMap to lookup the 
      # InstanceType value from the EnvironmentToInstanceTypeMap.
      # It references the EnvironmentType parameter provided to the template
      InstanceType: !FindInMap
        - EnvironmentToInstanceType # Map Name
        - !Ref EnvironmentType # Top Level Key
        - "InstanceType" # Second Level Key
```


This examples demonstrates how mapping is used in a CloudFormation template. It allows the creation of flexible templates. 

## Exercise - A simple map
Now it's your turn.
Lets add a Mappings section to our template. It will need to contain the instance type for `test` and `prod`. 
Update the `InstanceType` property of make use of this Mappings section.
The template has been updated to add an `EnvType` parameter.

* The instance types should be `t3.nano` and `t3.micro` for test and prod respectively.
The lab template is available at `code/40-cloudformation-features/05-lab04-Mapping.yaml`

{{%expand "Need a hint?" %}}
1. Create a `Mappings` section. 
  * Add a top level key of `InstanceType`.
  * Add a name-value pair for each of `test` and `prod`.
2. Update the `MyEC2Instance` resource.
  * Update the `InstanceType` property
  * Use the intrinsic function `Fn::FindInMap`
  * Reference the `EnvType` parameter

{{% /expand%}}

{{%expand "Expand to see the solution" %}}
```yaml
Mappings:
  Environment:
    InstanceType:
      test: t2.micro
      prod: m4.large

Resources:
  MyEC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: !Ref AmiID
      InstanceType: !FindInMap [Environment, InstanceType, !Ref EnvType]
      Tags:
        - Key: Name
          Value: !Join [ ' ', [ !Ref EnvType, Web Server ] ]

```

See `code/05/lab04-Mapping-Solution.yaml` for the full solution.
{{% /expand%}}

## Conclusion

In this lab, you used mappings to create flexible CloudFormation templates. Use Mappings to configure properties according to other parameters or pseudo parameters. A template can contain many maps, each with multiple top and second level keys. A Common use for a Mappings section is to provide different configurations depending on an environment type.

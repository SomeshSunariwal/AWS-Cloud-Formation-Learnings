# AWS-CloudFormation-Guide

## 1. Base Template

Starting of any template.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
Resources:
  MyEC2Instance: #Logical unique Identifier to each resource
    Type: AWS::ElastiCache::ReplicationGroup # Type of resource, Example : Redis Cache
    Properties:
```

In Properties you can the allowed properties for the particular resource

## 2. Parameters

This parameters let you configure you instance/resource parameter before creating stack.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template

Parameters:
  InstanceTypeParameter: # unique logical name or act as a variable
    Type: String # Type can be String, Number, List, CommaDelimitedList, AWS-Specific Parameter Types, SSM Parameter Types
    Default: t2.micro # this will be default selected in drop down
    AllowedValues:
      - t2.micro
      - m1.small
      - m1.large
    Description: Enter t2.micro, m1.small, or m1.large. Default is t2.micro. # this will be shown below just above of the selection box.
```

1.  if you want to limit the input values there is 'MaxValue' and 'MinValue' (only works for number type).
2.  if AllowValues tag is present then behave like a drop dropdown.
3.  if AllowValues tag is not present then behave like input field.

## 3. Mapping

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template

Parameter:
  LogicalValue:
    Type: String
    Default: Test1
    AllowValues:
      - Test1
      - Test2
      - Test3

Mapping:
  NewMap: # logical Unique Identifier or act as a variable name
    Test1:
      CacheNodeType: t2.micro
      NumCacheClusters: 1
    Test2:
      CacheNodeType: m1.small
      NumCacheClusters: 2
    Test3:
      CacheNodeType: m1.large
      NumCacheClusters: 3

Resources:
  MyEC2Instance: #Logical unique value to each resource
    Type: AWS::ElastiCache::ReplicationGroup # Type of resource, Example : Redis Cache
    Properties:
      CacheNodeType: !FindInMap [NewMap, !Ref LogicalValue, CacheNodeType]
      NumCacheClusters: !FindInMap [NewMap, !Ref LogicalValue, NumCacheClusters]
```

! FindInMap is a keyword to get a value from a map.

## 3. Intrinsic Functions

### 3.1. And Condition

Fn::And

`Short` !And

```yaml
Parameter:
  LogicalValue:
    Type: String
    Default: Test1
    AllowValues:
      - Test1
      - Test2
      - Test3
  Parameter2:
    Type: Number
    Default: 1
    MaxValue: 10
    MinValue: 1

  EncryptionValue:
    Type: String
    Default: true
    AllowValues:
      - "true"
      - "false"

Conditions:
  IsNumberOne: !Equals
    - !Ref Parameter2,
    - 1
  CanCreateResource: !And # Support Min 2 and Max 10 Conditions
    - !Condition : IsNumberOne
    - !Equals [!Ref LogicalValue, "Test1"] # and so on
  CanEncrypt: !Equals [!Ref EncryptionValue, "true"]

Resources:
  MyEC2Instance: #Logical unique value to each resource
    Type: AWS::ElastiCache::ReplicationGroup # Type of resource, Example : Redis Cache
    Condition: CanCreateResource # if this is then only this resource will be created.
    Properties:
      AtRestEncryptionEnabled: !If [CanEncrypt, "true", "false"]
```

### 3.2. Or Condition

Fn::Or

`Short` !Or

### 3.3. Not Condition

Fn::Not

`Short` !Not

```yaml
!Not [!Equals [!Ref VariableName, CompareWith]]
```

### 3.4. If Condition

Fn::If

```yaml
!If [Condition, !Ref If_Condition_True, "If_Condition_false"]
```

If condition is true then get the value from reference variable other wise assign the value 'If_Condition_false'

## 4. Stack Change Set

Add, Modify and Delete Resources in the already created stack.

1.  Select Stack which you want to modify.
2.  Click on `Stack Action`.
3.  Choose `Create change set for current stack`.
4.  Choose `Replace Current Template` if New Resource is added or removed or Choose `Use current template` if resource is modified.
5.  Click Next->Next->Next.
6.  Then Name a Change set and then Review Page will be shown to verify the Resources.
7.  Click `Execute` to deploy changes.

## 5. Pseudo Parameter

There are some pseudo parameters provided by AWS.

1.  `AWS::AccountId` : will return the account ID in which stack is creating.
2.  `AWS::NotificationARNs` : will return the notification ARN of the current stack if notification ARN is setup before creating stack.
3.  `AWS::NoValue` : will Removes the corresponding resource property when specified as a return value in the Fn::If intrinsic function.
4.  `AWS::Partition` : Returns the partition that the resource is in
5.  `AWS::Region` : will return the Region Name in which stack is creating.
6.  `AWS::StackId`: will return the stack current Stack ID
7.  `AWS::StackName` : Will return the current stack name
8.  `AWS::URLSuffix` : will return the URL suffix. Ex. amazonaws.com,

one way of use it to get the values and assign it to logical name or variable name.

```yaml
Condition:
  NeedEncryption: !If [!Equal [!Ref AccountId, "123123"], "yes", "AWS::NoValue"] # will return yes or remove that option property from the resource if they have.

Output: #shows the output
  AccountId:
    Value: !Ref "AWS::AccountId"

  NotificationARNs:
    Value: !Select [0, !Ref "AWS::NotificationARNs"]

  Partition:
    Value: !Ref "AWS::Partition"

  Region:
    Value: !Ref "AWS::Region"

  StackId:
    Value: !Ref "AWS::StackId"

  StackName:
    Value: !Ref "AWS::StackName"
```

## 6. User Data (AWS EC2)

This will run only when the instance is created Successfully. Its like running shell command after creating a EC2 instance.

```yaml
Resources:
  InstanceName:
    Type: AWS:EC2:Instance
    Properties:
      InstanceType: t2.micro
      UserData:
        Fn: Base64:
          !Sub |
          #!/bin/bash
          apt-get update -y
          apt-get install npm
          apt-get install create-react-app
```

Can be check in EC2 section

1. Click on newly launched EC2 instance.
2. Click on `Action` and then `Instance setting` and then `Edit user data`

Log of command execution can be seen using

```bash
cat /var/log/cloud-init-output.log
```

## 7. Output

This can be seen in output section the stack.

```yaml
Output:
  LogicalID: #
    Value: !Ref "AWS::StackName" #Mandatory Parameter
    Description: "Any Description you want to show"
    Export:
      Name: !Ref "AWS::StackName" # Should be unique through out the region
```

## 8. Some More Function

### 8.1 Fn::GetAttr or !GetAttr

Only get the attribute value if resource return the attribute value. can be check return section of every resource. Ex. https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html

```yaml
Fn::GetAttr [LogicalResourceName, attributeType]
```

```yaml
!GetAttr LogicalResourceName.attributeType
```

```yaml
Resources:
  InstanceName:
    Type: AWS:EC2:Instance
    Properties:
      InstanceType: t2.micro
```

way to get the value of instance type of Logical ID InstanceName

```bash
${InstanceName.InstanceType}
```

Same as

```yaml
!GetAttr InstanceName.InstanceType
```

### 8.2 !Sub

```yaml
UserData:
  Fn: Base64:
    !Sub |
    #!/bin/bash
    apt-get update -y
    mkdir /home/ubuntu/${AWS::StackName}
```

it will create a folder named with the current stack name after successful launch.

${AWS::StackName} will be substituted with AWS pseudo parameter values

With Custom Name

```yaml
UserData:
  Fn: Base64:
    !Sub
      - |
        #!/bin/bash
        apt-get update -y
        mkdir /home/ubuntu/${StackNewName}
      - { StackNewName : !Join [ "-", [ !Ref "AWS::StackName" , !Ref "AWS::Region" ] ] }
```

### 8.3 Fn::Join or !Join

join values like a concatenation

```yaml
!Join ["-", ["a", "b", "c"]]
```

`OR`

```yaml
!Join :
  - "-" # delimiter
  - - "a"
    - "b"
    - "c"
```

Result will be a-b-c

### 8.4 Fn::Select or !Select

Return single value from array or object.

```yaml
!Select [1, ["Test 1", "Test 2", "AWS::StackName", !Ref LogicalValue]]
```

it will return the index 1 value and Output will be "`b`".

If index is out of Bound and Null then it will give you `Stack Error`.

## 8.5 Fn::Split or !Split

```yaml
Fn::Split [delimiter, string]
```

`OR`

```yaml
!Split [delimiter, string]
```

Ex.

```yaml
!Select [0, !Split ["-", "a-b-c"]]
```

output will be "`a`"

### 8.6 Fn::FindInMap or !FindInMap

```yaml
Fn::FindInMap [LogicalID, TopLevelKey, SecondLevelKey]
```

`OR`

```yaml
!FindInMap [LogicalID, TopLevelKey, SecondLevelKey]
```

Ex.

```yaml
Mapping:
  MapName:
    LogicalID1:
      ValueName: 1
      ValueID: 2
    LogicalID2:
      ValueName: 3
      ValueID: 4

!FindInMap [MapName, LogicalID1, ValueName]
```

will return "`1`"

Best Example can be seen above in `Section 3`.

### 8.7 Fn::GetAZs or !GetAZs

Will return availability zone within region.

Ex.

```Yaml
!GetAZs : "us-east-1"
```

will return : `["us-east-1a", "us-east-1b", "us-east-1c"]`.

#### `Error Case`

```yaml
!Select [0, [!GetAZs !Ref "AWS::Region"]]
```

This will give Error because `we cannot use short form of intrinsic function conjugately`.

Can be used as

```yaml
!Select
- 1
- !GetAZs
  - !Ref "AWS::Region"
```

### 8.8 Fn::ImportValue

Cross Stack Reference

Ex. Stack 1 output

```yaml
Output:
  StackNameExport: # Should be unique ins entire region
    Value: !Ref "AWS::StackName"
    Description: "Return Stack Name from this stack"
    Export:
      Name: !Ref "AWS::StackName"
```

Stack 2

```yaml
ClusterName : !ImportValue : StackNameExport
```

it will set the cluster name as stack name.

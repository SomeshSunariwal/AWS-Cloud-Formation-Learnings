# AWS-CloudFormation-Learnings

## 1. Base Template

Starting of any template.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
Resources:
  MyEC2Instance: #Logical unique value to each resource
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
  NewMap: # logical name or act as a variable name
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

### 1. And Condition

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

Fn::Or

`Short` !Or

Fn::Not

`Short` !Not

```yaml
!Not [!Equals [!Ref VariableName, CompareWith]]
```

Fn::If

```yaml
!If [Condition, !Ref If_Condition_True, "If_Condition_false"]
```

If condition is true then get the value from reference variable other wise assign the value 'If_Condition_false'

# AWS-Cloud-Formation-Learnings

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

1.  if you want to limit the input values there is "MaxValue" and "MinValue" (only works for number type).
2.  if AllowValues tag is present then behave like a drop dropdown.
3.  if AllowValues tag is not present then behave like input field.

## 3. Mapping

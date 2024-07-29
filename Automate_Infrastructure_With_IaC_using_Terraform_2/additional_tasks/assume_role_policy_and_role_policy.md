# Difference Between Assume Role Policy and Role Policy

## Introduction

In AWS Identity and Access Management (IAM), roles are an essential part of managing permissions and access within an AWS account. Two key concepts often associated with roles are the `Assume Role Policy` and the `Role Policy`. Understanding the difference between these two policies is crucial for managing security and access controls effectively.

## Assume Role Policy

### Definition

An `Assume Role Policy`, also known as a `Trust Policy`, is a JSON policy document attached to an IAM role that specifies which entities (users, groups, services, or other roles) are allowed to assume the role. This policy defines the trust relationship between the role and the entity that will assume it.

Key Points

-	__Purpose:__ Defines who can assume the role.

-	__Format:__ JSON document.

-	__Components:__
    -	__Principal:__ Specifies the entity that is allowed to assume the role (e.g., another IAM role, AWS service, IAM user, etc.).

    -	__Action:__ Typically includes the sts:AssumeRole action.

    -	__Condition:__ Optional conditions that must be met for the assumption to be allowed.

Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/OtherRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
In the example above, the Assume Role Policy allows the role OtherRole to assume the role to which this policy is attached.


## Role Policy

### Definition

A `Role Policy`, also known as an Inline Policy or an Attached Policy, is a JSON policy document that defines what actions and resources the role is allowed or denied to access. This policy grants permissions to the role to perform specific actions on specified AWS resources.

### Key Points

-	__Purpose:__ Defines what the role can do once it has been assumed.

-	__Format:__ JSON document.

-	__Components:__
	-	__Action:__ Specifies the actions that are allowed or denied (e.g., s3:ListBucket, ec2:StartInstances).

	-	__Resource:__ Specifies the AWS resources that the actions apply to (e.g., an S3 bucket ARN, an EC2 instance ARN).

	-	__Effect:__ Indicates whether the actions are allowed (Allow) or denied (Deny).

	-	__Condition:__ Optional conditions under which the permissions apply.

Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::example-bucket",
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```
In this example above, the Role Policy allows the role to list and get objects from the specified `S3 bucket` example-bucket.

### Key Differences

|  Aspect | Assume Role Policy | Role Policy |
|:-----------:|:---------:|:----------------:|
| Purpose | Specifies who can assume the role | Specifies what actions and resources the role can access |
| Entities Involved | Defines trust relationship with IAM users, roles, or services | Defines permissions for the role itself |
| Actions Included | Typically includes sts:AssumeRole | Includes various AWS service actions (e.g., s3:ListBucket, ec2:StartInstances) |
| Scope | Who can assume the role | What the role can do once assumed |
| Example Use Case | Allowing an EC2 instance to assume a role | Granting a role permissions to access S3 buckets and EC2 instances |


## Conclusion

Understanding the difference between `Assume Role Policy` and `Role Policy` is fundamental for managing roles in AWS IAM. The `Assume Role Policy` focuses on establishing `trust` and specifying who can assume the role, while the `Role Policy` defines the permissions and access control for the role once it is assumed. Together, these policies provide a robust mechanism for managing access and permissions in an AWS environment.









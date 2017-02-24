---
date: 2017-02-20T06:39:45+13:00
title: CloudFormation Resources
---

## Introduction

The `aws-cloudformation` role included in the framework contains a CloudFormation resources template that is designed to define the following AWS resources:

- S3 bucket for CloudFormation templates
- S3 bucket for Lambda functions used to support CloudFormation custom resources
- Key Management Service (KMS) Key used for secrets management in CloudFormation stacks

By default, without any custom configuration, the CloudFormation resources template will create each of the above resources.

The template currently includes some optional configuration parameters as described below:

- `config_cfn_lambda_remote_accounts` - optional list of accounts that the S3 bucket for Lambda functions will be shared with.
- `config_cfn_kms_admins` - optional list of IAM roles that have administration level access to manage the KMS key.  By default the **admin** role created by the security resources stack is granted this access.
- `config_cfn_kms_encrypters` - optional list of IAM roles that have permission to encrypt ciphertext using the KMS key.  By default the **admin** role created by the security resources stack is granted this access.
- `config_cfn_kms_decrypters` - optional list of IAM roles that have permission to decrypt ciphertext using the KMS key.  By default the **admin** role created by the security resources stack is granted this access.
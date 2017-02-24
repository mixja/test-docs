---
date: 2017-02-18T17:15:17+13:00
title: Tutorial
---

## Introduction

This tutorial provides a walkthrough of creating all of your AWS resources from scratch for a given AWS account, using the AWS deployment framework. 

In this tutorial you will be configuring a number of AWS resources:

- [Security Resources]({{< relref "tutorial/security-resources/index.md" >}})
- [CloudFormation Resources]({{< relref "tutorial/cloudformation-resources/index.md" >}})
- [Network Resources]({{< relref "tutorial/network-resources/index.md" >}})
- [EC2 Container Registry (ECR) Resources]({{< relref "tutorial/ecr-resources/index.md" >}})
- [Web Proxy]({{< relref "tutorial/web-proxy/index.md" >}})
- [Intake API Application]({{< relref "tutorial/intake-api/index.md" >}})
- [Intake Accelerator Application]({{< relref "tutorial/intake-accelerator/index.md" >}})

Each of the above consists of the following:

- Ansible playbook
- CloudFormation template that defines all AWS resources
- One or more environments, each environment ultimately implemented as a CloudFormation stack
- Git repository

## Prerequisites

This tutorial assumes you are starting from an empty AWS account and have access to the root account.

Each of the playbooks described in the [Introduction]({{< relref "tutorial/index.md#introduction" >}}) are already defined as existing Git repositories, so you will require access to each repository.

Each playbook also assumes that your local environment is configured with appropriate permissions to create the various AWS resources defined in each playbook.  For example, if your environment specification in a given playbook targets account ID 123456789012, then it is expected that your local AWS environment is configured apporpriate to access that environment.

For example assuming the following local AWS environment setup:

#### ~/.aws/credentials:

```
[caintake-users]
aws_access_key_id = xxxxxxx
aws_secret_access_key = xxxxxxx
```

#### ~/.aws/config:

```
[profile caintake-sandpit]
source_profile=caintake-users
role_arn=arn:aws:iam::123456789012:role/admin
mfa_serial=arn:aws:iam::391236350805:mfa/JustinMenga
region=us-west-2
```

Assuming the user `JustinMenga` in the `caintake-users` account has permissions to assume the `admin` role in the account `123456789012` and all playbooks are targetting the account `123456789012`, you would run all playbooks with the following environment configuration:

```bash
$ export AWS_PROFILE=caintake-sandpit
$ ansible-playbook site.yml -e env=dev
...
...
```


## Forking the Starter Repository

Although each of the playbooks described in the [Introduction]({{< relref "tutorial/index.md#introduction" >}}) are already defined as existing Git repositories, it is useful to understand how to create a playbook from scratch that implements the AWS deployment framework and methodology described in this tutorial.

A starter repository is published at https://github.com/casecommons/aws-starter.git, which you can use to fork a new repository, and follow the instructions in the [README](https://github.com/casecommons/aws-starter/README.md) to setup your playbook correctly.
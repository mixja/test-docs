---
date: 2017-02-18T23:54:48+13:00
title: Creating Security Resources
---

## Introduction

The `aws-cloudformation` role included in the framework contains a security resources template that is designed to define the following security resources:

- IAM roles
- IAM groups
- ACM certificates

By default, without any custom configuration, the security template creates the following resources:

- An **admin** IAM role that has the AWS **AdministratorAccess** managed policy attached and trusts your AWS account and an optional list of other AWS accounts to assume the role
- An **Administrators** IAM group that has an IAM policy attached that permits members of this group the ability to assume the **admin** role.
- A **Users** IAM group that has an IAM policy attached that enforces a requirement for multi-factor authentication (MFA) for all AWS operations except for a minimal set of IAM operations that allow a user to register/resync his/her MFA device and change his/her password.

## Creating a temporary user

The security resources stack is the first stack you will create in a given AWS account, and requires a temporary administrative user to be created with an API access key configured to run the security resources playbook.

> These instructions assume you are configuring a new account and have logged in to the account using root credentials to perform initial setup.

1.  In the IAM dashboard, select  **Users** on the left panel and click **Add User**.

    ![IAM Dashboard](/images/iam-dashboard.png)

2.  Specify an appropriate user name, select the option to enable **Programmatic access**, and click **Next: Permissions**.

    ![Add User](/images/iam-add-user.png)

3.  Select the **Attach existing policies directly** option and type **AdministratorAccess** in the **Filter** input.  Ensure the **AdministratorAccess** policy is selected and then click **Next: Review**.

    ![Set Permissions](/images/iam-set-permissions.png)

4.  Click on the **Create User** button.

    ![Create User](/images/iam-create-user.png)

5.  Click on the **Show** link to display the secret access key for the newly created user.

    ![Access Key](/images/iam-access-key.png)

6.  Copy the **Access key ID** and **Secret access key** values which you will need to configure the environment.  Once you have copied these values, click on the **Close** button.

    ![Show Access Key](/images/iam-show-access-key.png)

7.  Open a new shell prompt and export the access key ID and secret access key values as demonstrated below.

    ```bash
    $ export AWS_ACCESS_KEY_ID=AKIAIRJGYMTD252YWCTQ
    $ export AWS_SECRET_ACCESS_KEY=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
    $ export AWS_DEFAULT_REGION=us-west-2
    ```

## Installing the playbook locally

The security resources playbook is located at https://github.com/casecommons/caintake-security.git.  In this section we will add a new environment called **demo** to the playbook, which will be used to create security resources in our new account.

First clone the repository locally and change to the `caintake-security` folder:

```bash
$ git clone git@github.com:casecommons/caintake-security.git
  Cloning into 'caintake-security'...
  remote: Counting objects: 32, done.
  remote: Total 32 (delta 0), reused 0 (delta 0), pack-reused 32
  Receiving objects: 100% (32/32), 4.78 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (7/7), done.
$ cd caintake-security
$ tree
.
├── README.md
├── ansible.cfg
├── group_vars
│   ├── all
│   │   └── vars.yml
│   └── non-prod
│       └── vars.yml
├── inventory
├── roles
│   └── requirements.yml
└── site.yml
```

Review the `roles/requirements.yml` file, which specifies the location and version of Ansible roles that the playbook relies on:

```python
- src: git@github.com:Casecommons/aws-cloudformation.git
  scm: git
  version: 0.7.0
  name: aws-cloudformation
- src: git@github.com:Casecommons/aws-sts.git
  scm: git
  version: 0.1.2
  name: aws-sts
```

Install the roles using the `ansible-galaxy` command as demonstrated below:

```bash
$ ansible-galaxy install -r roles/requirements.yml --force
- extracting aws-cloudformation to /Users/jmenga/Source/casecommons/caintake-security/roles/aws-cloudformation
- aws-cloudformation was installed successfully
- extracting aws-sts to /Users/jmenga/Source/casecommons/caintake-security/roles/aws-sts
- aws-sts was installed successfully
```

Notice that this installs the roles locally into the `roles` folder.

{{% note title="Note" %}}
You only need to run the `ansible-galaxy install` command as demonstrated above in the following scenarios:

- Cloning the repository for the first time
- Updating the locally installed roles to a newer version
- Resetting the locally installed roles to their original state

Using the `--force` flag ensures the role will be overwritten regardless of whether or not the role is currently present locally.
{{% /note %}}

Review the `group_vars/all/vars.yml` file, which contains global settings for the security resources playbook:

```python
# Stack Settings
cf_stack_name: security-resources
cf_stack_template: "templates/security.yml.j2"
cf_stack_tags:
  org:business:owner: CA Intake
  org:business:product: Security
  org:business:severity: Severity1
  org:tech:environment: "{{ env }}"
  org:tech:contact: pema@casecommons.org
  
# CloudFormation settings
# This sets a policy that disables updates to existing resources
# This requires you to explicitly set the cf_disable_stack_policy flag to true when running the playbook
cf_stack_policy:
  Statement:
  - Effect: "Deny"
    Action: "Update:*"
    Principal: "*"
    Resource: "*"
```

Notice that the `cf_stack_template` setting references `templates/security.yml.j2`, which is a relative reference to a template included within the `aws-cloudformation` role.

Finally review the `site.yml` file, which contains the main playbook:

```python
- name: Assume Role
  hosts: "{{ env }}"
  gather_facts: no
  roles:
    - aws-sts

- name: Stack Deployment
  hosts: "{{ env }}"
  environment: "{{ sts_creds }}"
  roles:
    - aws-cloudformation
```

Notice the playbook is very simple with two plays defined:

- Assume Role - this calls the `aws-sts` role and assumes the role defined by the `sts_role_arn` setting for a given environment.
- Stack Deployment - this calls the `aws-cloudformation` role and creates a CloudFormation stack for a given environment.  Notice that we inject `sts_creds` into the environment, which contains the temporary session credentials obtained via the previous Assume Role play.

In both plays, notice that the `hosts` variable references the `env` variable, which we must always supply at runtime:

```
$ ansible-playbook site.yml -e env=<environment-name>
```

An interesting aspect of the framework is that you generally don't need to modify the primary `site.yml` playbook, unless you want to perform some custom pre-deployment or post-deployment actions.

## Defining a new environment

In this section we will add a new environment called **demo** to the playbook, which will be used to create security resources in our new account.

First add **demo** as a new environment in the `inventory` file:

```toml
[non-prod]
non-prod ansible_connection=local

[demo]
demo ansible_connection=local
```

Next create a file called `group_vars/demo/vars.yml`, which will hold all environment specific configuration for the **demo** environment:

```bash
$ mkdir -p group_vars/demo
$ touch group_vars/demo/vars.yml
```

Finally add the following settings to `group_vars/demo/vars.yml`, replacing `625916301437` with the account ID of the **demo** account:

```python
# STS role to assume
sts_role_arn: "arn:aws:iam::625916301437:role/admin"

# List of accounts to trust to assume the admin role
# This example adds the caintakeusers account as a trusted entity
config_iam_admin_accounts:
  - 391236350805
```

The `sts_role_arn` setting defines the Amazon Resource Name (ARN) of the IAM role the `aws-sts` Ansible role (take care not to confuse IAM and Ansible roles) will attempt to assume.  This is a very important setting, as it establishes the target account that the playbook will create a CloudFormation stack and associated resources in.

The `config_iam_admin_accounts` setting is specific to the security resources template, and allows you to configure other AWS accounts as trusted entities that are allowed to assume the **admin** role that is automatically created as part of the security resources stack.

This supports a common security pattern where all AWS users are defined in a central **"users"** account and then assume roles into **"resource"** accounts.  

In this example, our new account is considered a **"resource"** account and we are configuring the admin role to trust users from the account ID **391236350805**.

{{% note title="Note" %}}
Adding a remote account as a trusted entity does not automatically allow all users from the remote account to assume the role.

Users in the remote account must be granted permissions to assume the role - hence we are delegating control of which users can assume the admin role to the trusted account.
{{% /note %}}

## Running the playbook

The security template in the `aws-cloudformation` role will create the following resources by default:

- An IAM role named **admin**
- An IAM group named **Administrators** that grants members the ability to assume the **admin** role
- An IAM group named **Users** that requires users to authenticate using multi-factor authentication (MFA)

Let's now run the playbook to create these resources.  

> These instructions assume you have configured your local environment with the AWS access key and AWS secret access key created earlier in  [Creating a temporary user]({{< relref "security-resources/index.md#creating-a-temporary-user" >}})

First run the Ansible playbook targeting the `demo` environment as demonstrated below:

```bash
$ ansible-playbook site.yml -e env=demo

PLAY [Assume Role] *************************************************************

TASK [aws-sts : set_fact] ******************************************************
ok: [demo]

TASK [aws-sts : checking if sts functions are sts_disabled] ********************
skipping: [demo]

TASK [aws-sts : setting empty sts_session_output result] ***********************
skipping: [demo]

TASK [aws-sts : setting sts_creds if legacy AWS credentials are present (e.g. for Ansible Tower)] ***
skipping: [demo]

TASK [aws-sts : assume sts role] ***********************************************
fatal: [demo]: FAILED! => {"changed": false, "cmd": "aws sts assume-role --role-arn=\"arn:aws:iam::625916301437:role/admin\" 
--role-session-name=\"adminSession\"", "delta": "0:00:02.143031", "end": "2017-02-19 14:42:24.227270", "failed": true, 
"rc": 255, "start": "2017-02-19 14:42:22.084239", "stderr": "\nAn error occurred (AccessDenied) when calling the AssumeRole 
operation: Not authorized to perform sts:AssumeRole", "stdout": "", "stdout_lines": [], "warnings": []}

PLAY RECAP *********************************************************************
demo                       : ok=1    changed=0    unreachable=0    failed=1 
```


Notice that the operation fails, and this is because we have a chicken-in-egg scenario where our the `aws-sts` Ansible role in our playbook is attempting to assume the IAM role defined by the `sts_role_arn` setting in our `groups_vars/demo/vars.yml` file:

```python
# STS role to assume
sts_role_arn: "arn:aws:iam::625916301437:role/admin"
...
...
```

This **admin** role does not exist yet, as it is the job of the security playbook to create the role, creating a chicken-in-egg scenario.

To overcome this we can set a flag `sts_disable` to `true`, which disables the `aws-sts` role:

```bash
$ ansible-playbook site.yml -e env=demo -e sts_disable=true

PLAY [Assume Role] *************************************************************

TASK [aws-sts : set_fact] ******************************************************
ok: [demo]

TASK [aws-sts : checking if sts functions are sts_disabled] ********************
ok: [demo] => {
    "msg": "Skipping STS functions as sts_disabled is defined"
}

...
...

TASK [aws-cloudformation : configure application stack] ************************
changed: [demo]

TASK [aws-cloudformation : get stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : set stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : debug] **********************************************
skipping: [demo]

TASK [aws-cloudformation : S3 Template URL] ************************************
skipping: [demo]

TASK [aws-cloudformation : enable current stack policy] ************************
skipping: [demo]

TASK [aws-cloudformation : fail] ***********************************************
skipping: [demo]

TASK [aws-cloudformation : delete application stack] ***************************
skipping: [demo]

PLAY RECAP *********************************************************************
demo                       : ok=19   changed=1    unreachable=0    failed=0
```

This time the playbook succeeds, pausing on the `configure application stack` task for a couple of minutes whilst the CloudFormation stack is created (notice this task completes with a `changed` status), and then completing successfully once the stack is created.

If you navigate to the CloudFormation dashboard, notice that a new stack called **security-resources** has been created:
![Security Resources Stack](/images/security-resources-stack.png)

In the IAM dashboard we can verify that our **admin** role has been created and if we select the role and open the **Trust Relationships** tab, we can see that both the local account (account ID **625916301437**) and remote **"users"** account (account ID **391236350805**) are trusted entities that can assume the **admin** role:
![IAM Admin Role](/images/iam-admin-role.png)

Because the **admin** role has now been created, you should be able to re-run the playbook without the `sts_disable` flag:

```bash
$ ansible-playbook site.yml -e env=demo

PLAY [Assume Role] *************************************************************

TASK [aws-sts : set_fact] ******************************************************
ok: [demo]
...
...

TASK [aws-cloudformation : configure application stack] ************************
ok: [demo]

TASK [aws-cloudformation : get stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : set stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : debug] **********************************************
skipping: [demo]

TASK [aws-cloudformation : S3 Template URL] ************************************
skipping: [demo]

TASK [aws-cloudformation : enable current stack policy] ************************
skipping: [demo]

TASK [aws-cloudformation : fail] ***********************************************
skipping: [demo]

TASK [aws-cloudformation : delete application stack] ***************************
skipping: [demo]

PLAY RECAP *********************************************************************
demo                       : ok=20   changed=0    unreachable=0    failed=0
```

This time the playbook can successfully assume the **admin** role (because our temporary user has administrative credentials and the role trusts the local AWS account), and notice that the **configure application stack** completes with no change, as the CloudFormation stack already exists and no configuration changes have been made.

Back in the IAM console, you should also be able to see that an **Administrators** group and **Users** group have been created:

![IAM Groups](/images/iam-groups.png)

At this point, you can remove the temporary administrative user [we created earlier]({{< relref "security-resources/index.md#creating-a-temporary-user" >}}) by selecting the user in the IAM users section, clicking **Delete users** and confirming the delete operation:
![Delete User](/images/iam-delete-user.png)
![Confirm Delete User](/images/iam-confirm-delete-user.png)

You should also now clear the previous temporary user credentials from your local shell:

```bash
$ unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_DEFAULT_REGION
```

## Configuring the users account

In the previous section we added a **"users"** account as a trusted entity that is allowed to assume the **admin** role.

Before users in the **"users"** account can assume the role, users within that account must be explicitly granted permission to assume the **admin** role.

First login to the **"users"** account and navigate to the IAM dashboard.  In this example, we are logging into the **caintakeusers** account, which is the account we configured as a trusted entity for our new account's IAM **admin** role:

![Logging in to the Users Account](/images/login-users.png)

Navigate to the IAM dashboard Groups section and add a new group called DemoAdmins:

![Create Group](/images/iam-create-group.png)

![Set Group Name](/images/iam-set-group-name.png)

When prompted to attach a policy, click on **Next Step** to skip this section and then confirm creation of the group:

![Skip Group Policy](/images/iam-skip-group-policy.png)

![Confirm Group](/images/iam-create-group-confirm.png)

Select the **Permissions** tab of the newly created group, expand **Inline Policies** and create a new inline policy by clicking the **click here** link:

![Confirm Group](/images/iam-group-inline-policy.png)

Choose the **Custom Policy** option and click **Select**:

![Confirm Group](/images/iam-group-custom-policy.png)

Copy the following policy document snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1480629742000",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "arn:aws:iam::625916301437:role/admin"
            ]
        }
    ]
}
```

Name the policy **AssumeDemoAdminRole**, paste the policy document snippet into the **Policy Document** window and then validate and apply the policy:

![Confirm Group](/images/iam-policy-document.png)

![Confirm Group](/images/iam-group-policy-attached.png)

Finally we need to attach users go the newly created group:

![Confirm Group](/images/iam-add-users-group.png)

![Confirm Group](/images/iam-select-users.png)

![Confirm Group](/images/iam-users-added.png)

The instructions above demonstrate how to manually configure the **"users"** account, however ideally you would configure these settings in a security playbook targeting the **users** account environment, run the playbook and then add users to the group via the AWS console:

```python
# Snippet from group_vars/<users-environment>/vars.yml
...
...
config_iam_groups:
  DemoAdmins:
    Policies:
      - PolicyName: AssumeDemoAdminRole
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Effect: "Allow"
              Action:
              - "sts:AssumeRole"
              Resource: "arn:aws:iam::625916301437:role/admin"
...
...
```

## Verifying role assumption

To wrap up we can now verify that the user account(s) we just added to the **DemoAdmins** group in our central "users" account can assume the demo account **admin** role.

The following snippet creates a profile that will assume this role in the local `~/.aws/config` file:

```toml
...
[profile caintake-demo-admin]
source_profile=caintake-users
role_arn=arn:aws:iam::625916301437:role/admin
mfa_serial=arn:aws:iam::391236350805:mfa/JustinMenga
region=us-west-2
...
```

With this profile in place, you can now configure the `AWS_PROFILE` environment variable and verify role assumption.  As before, you are able to assume the role and the playbook completes successfully with no change, as there has been no change to the security playbook configuration settings:

```bash
$ export AWS_PROFILE=caintake-demo-admin
$ ansible-playbook site.yml -e env=demo

PLAY [Assume Role] *************************************************************

TASK [aws-sts : set_fact] ******************************************************
ok: [demo]
...
...

TASK [aws-cloudformation : configure application stack] ************************
ok: [demo]

TASK [aws-cloudformation : get stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : set stack facts] ************************************
ok: [demo]

TASK [aws-cloudformation : debug] **********************************************
skipping: [demo]

TASK [aws-cloudformation : S3 Template URL] ************************************
skipping: [demo]

TASK [aws-cloudformation : enable current stack policy] ************************
skipping: [demo]

TASK [aws-cloudformation : fail] ***********************************************
skipping: [demo]

TASK [aws-cloudformation : delete application stack] ***************************
skipping: [demo]

PLAY RECAP *********************************************************************
demo                       : ok=20   changed=0    unreachable=0    failed=0
```
---
date: 2016-03-08T21:07:13+01:00
title: Cloud Deployment Framework
type: index
weight: 0
---

The Cloud Deployment Framework (hereafter referred to simply as the **framework**) is a methodology and collection of tools that helps organizations:

- Deploy cloud infrastructure and services in a fully automated fashion
- Support configuration management, version control and change control
- Create, update and destroy complete environments on-demand
- Establish a standard suite of cloud resources and services to enable product and application teams to develop, build and deploy services faster

The framework currently supports Amazon Web Services (AWS) although there is no reason why the same principles cannot be applied to other cloud providers.

## Overview

Amazon Web Services (AWS) provides an extensive and sophisticated range to building blocks to help organizations build digital platforms, applications and products in the cloud.

Today AWS provides almost [100 products and services](https://aws.amazon.com/products/), and this list is ever increasing as startups and enterprises alike continue to aggressively adopt the cloud to realize its benefits of speed, flexibility and pay-per-use model.

Making sense of all of these technologies is challenging, yet alone understanding how to automate the deployment of these services, even if you are only consuming a handful of AWS services.

AWS provides a management product called [CloudFormation](https://aws.amazon.com/cloudformation/), which allows you to create, update and destroy related sets of AWS resources using a data-driven ***infrastructure as code*** format.  CloudFormation provides you with the building blocks of declaratively defining your AWS infrastructure in a reusable and repeatable manner, but still requires you to provide wrap own orchestration and automation solution, which is where we introduce [Ansible](https://www.ansible.com).

[Ansible](https://www.ansible.com) is a popular open-source configuration management tool that traditionally has competed with the likes of Puppet, Chef and Salt to perform configuration management of server infrastructure.  Ansible uses a task-oriented workflow that provides sequential processing of a set of tasks, making it well suited for orchestration scenarios where strong control over dependencies and the order of execution is required.

This document describes a methodology and framework for deploying AWS resources based upon the use of AWS CloudFormation and Ansible, which we will refer to throughout this document as the **Cloud Deployment Framework**, or simply the **framework**.

## Features

The Cloud Deployment Framework offers the following features:

- **Defines complete environments** - basically anything you can describe in a CloudFormation template, whether it be a simple S3 bucket, all of the networking resources in your account, or a complex application platform that includes CloudFront CDN, Route 53 DNS, application load balancer, EC2 autoscaling group, RDS database and EC2 Container Service resources.

- **Supports multiple environments** - full support for multiple environments (e.g. dev, test, staging, production) that can be distributed across one or multiple AWS accounts.

- **Infrastructure as code** - provides a simple self-contained folder-based structure that allows you to leverage the power of Git for version control and code review, and easily integrates with Continuous Delivery systems such as Jenkins for automated deployments.

- **Reusable templates and patterns** - the framework includes a number of reusable templates that adopt best practice patterns for creating your AWS infrastructure.

- **Flexible and powerful** - the framework allows you to leverage pre-defined templates, create your own simple static CloudFormation templates or create sophisticated dynamic CloudFormation templates using Ansible's Jinja templating engine.

## Architecture

The architecture of the cloud deployment framework is modular and flexible.

At the lowest level, the framework is based upon an Ansible playbook structure that allows you to define global and environment specific settings, and allows you to generate and transform configuration artefacts, such as CloudFormation templates, using the Jinja2 templating framework built into Ansible.

The next level leverages Ansible roles to add packaging and reusabling plugins to the overall architecture.  For example to support AWS deployments, two key Ansible roles can be used:

- `aws-sts` role - provides assume role functionality, allowing you to assume different roles for different environments.
- `aws-cloudformation` role - provides a generic approach to configuring and deploying CloudFormation templates and stacks.

THe architecture is such that you can write your own roles that can target different use cases and/or different cloud providers.

At a top level the architecture is very much dependent on the roles you are leveraging.  For example, the `aws-cloudformation` role allows you to define any generic CloudFormation template you want, but also includes a suite of predefined templates that help bootstrap an AWS account from scratch.

The architectural approach allows users to 

### Ansible

Although Ansible supports an extensive range of modules that can provision many different AWS resources, in the context of the methodology presented we utilise Ansible for three key features:

- Task oriented workflow - Ansible's task-oriented and extensible workflow makes it powerful for orchestrating complex workflows.  Ansible roles allow us to create reusable workflows that we can then combine together to orchestrate a complex set of end-to-end provisioning tasks and outcomes.

- Template generation - Ansible supports powerful template generation capabilities based upon the popular [Jinja templating engine](http://jinja.pocoo.org).  By applying template generation techniques to CloudFormation templates We can use these template generation capabilities to create 

Ansible also supports a powerful templating module based upon the popular [Jinja templating language](http://jinja.pocoo.org), Ansible provides extensive support for AWS services, including CloudFormation, and we can use CloudFormation to define our AWS resources, and then use Ansible to orchestrate 

By combining the strengths of Ansible and CloudFormation, you can create a very powerful methodology and framework for creating, updating and destroying your AWS infrastructure.  

We now present a framework and methodology that we will refer to simply as the **framework** or **methodology** - 



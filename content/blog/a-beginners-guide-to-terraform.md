+++
author = "Eduardo Rodriguez"
title = "A beginner's guide to Terraform"
date = "2024-09-12"
description = "Or how to perform your first cloud deployment with Terraform"
tags = [
    "terraform",
    "iac"
]
toc = true
+++

This article offers an introduction into writing _Infrastructure as Code_ (IaC) with **Terraform**.

## Why use Terraform?

Before I started working at [they consulting](https://they-consulting.de) I used to do all my cloud deployments manually in the web console of the cloud providers that I used, be it AWS, DigitalOcean, Hetzner or Vultr.

Most of the time I would be deploying some kind of web service into a virtual machine (VM), but since I hadn't properly learnt to use IaC the whole process would require a lot of my own time and effort.
A very basic deployment process would require me to:

1. Log into the web page of the cloud provider.
2. Use their web UI (console) to create a virtual machine. (This step is particularly cumbersome since every time I would have to go through the forms required to setup a VM and every time I would fill-in the same information: `name`, `size`, `operating system`, `SSH keys`, etc.)
3. Wait for the VM to be created...
4. Log in to the VM with SSH.
5. `git clone` whatever repository I'd wanted to deploy.
6. Manually start the web service.

As you can see these are a lot of steps that do not change much between different deployments, at most the code being cloned in step #5 has changed.

Terraform (or any other IaC platform for that matter) let's us encapsulate all these repetitive steps in actual **code** and takes care of running all these steps programatically, so that we can just sit back, relax and see Terraform do the hard work.

## How does Terraform work?

In order to use Terraform you should have an understanding for three main concepts: **providers**, **resources** and **state**.

### Providers

**Providers** are the cloud platforms (AWS, Azure, DigitalOcean) that have created and maintain a library for Terraform that allows users to indirectly interact with their APIs in order to create, change or destroy some of the cloud products that they offer.

You can find all the providers currently supported by Terraform in the [official Terraform docs](https://registry.terraform.io/browse/providers).

When creating a Terraform deployment you would normally start by creating a `providers.tf` file in which you configure the provider(s) that you will be using in your deployment.

For example, if you want to work with AWS, you can find in the [AWS provider documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) how to properly setup your `providers.tf` file:

```hcl
# File: providers.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

Here you can define with which version of the AWS provider you want to work and in which AWS region your resources should be deployed.

### Resources

**Resources** are the actual cloud products or services that you are trying to configure/setup/deploy/destroy in whatever cloud provider you have chosen.

So, for example, if you are trying to create a VM in AWS (an `EC2 instance`) you would describe all the options with which you can configure the `EC2 instance` in the Terraform code:

```hcl
# File: ec2.tf

resource "aws_instance" "ec2_resource_name" {
  ami           = data.aws_ami.amzn-linux-2023-ami.id
  instance_type = "t4g.nano"

  tags = {
    Environment = "dev"
  }
}
```

If you are working with one of the big cloud providers (AWS, Azure, GCP), it is quite likely that you will be capable of configuring your resources with Terraform in the same way as you are capable of manually configuring them in the web console of the cloud providers.

In this particular code we can see that I am creating a resource of type `aws_instance` (an `EC2 instance`) and I am naming this particular resource `ec2_resource_name`.
Then, I am defining some configuration options for the `EC2 instance` like the Linux distribution that it should use (`ami`), the type of instace that I would like to use (`instance_type`), etc.

> **Protip**: You usually want to start by actually creating your cloud resources manually in the web console, before writing any Terraform code.
> You will get a feeling for the kind of resources that you'll need for your deployment. <br>
> Later, after you have successfully created your resources with the console, you can start migrating the deployment to Terraform code. <br> <br>
> Especially when you are working with a lot of complicated resources with which you might have not worked before, it might be very overwhelming to start right away writing Terraform code.

### State

So, you have configured your provider and you have written some Terraform files describing the resources that you want to create, now you can actually go ahead and use Terraform to create those resources in the cloud.

In the case of AWS, you will have to create an account and setup the [AWS Command Line Interface (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) in your machine.
After properly configuring AWS CLI Terraform should be capable of using your AWS profile to create resources in AWS.

Now open the command line, go to the directory where you created the `providers.tf` and `ec2.tf` files and run `terraform init` (you should have previously installed terraform in your computer).

`terraform init` locally downloads all the files required to interact with the providers and initializes a working directory with Terraform configuration files. If the command succeeds, you can follow it up with `terraform plan`.

#### Terraform plan

From the [terraform plan documentation](https://developer.hashicorp.com/terraform/cli/commands/plan):

> The terraform plan command creates an **execution plan**, which lets you preview the changes that Terraform plans to make to your infrastructure. [...] <br>
>
> - [Terraform] reads the **current state** of any already-existing remote objects to make sure that the Terraform **state** is up-to-date.
> - Compares the current configuration to the **prior state** and noting any differences.
> - Proposes a set of change actions that should, if applied, make the remote objects match the configuration.

In simpler words, Terraform knows which resources, with which options have been previously deployed (this is the `state`).

If no resources have been previously created, Terraform will create all your resources from scratch. But if you have already created some resources within the same Terraform project, Terraform will realize that there are already some resources deployed (there is a **prior state**), so it will check your `*.tf` files in order to see what changes to the deployed resources you have defined in you `*.tf` files.

For instance, you might want to change the `Tags` in your previously created `EC2 instance`. If that's the case, Terraform will see if it is capable of modifying the `EC2 instance` that is already running, and if not, it will destroy it and create a new one.

You can check the output of the `terraform plan` command in order to see which resources would be created/destroyed/modified by Terraform.
The `terraform plan` command is not actually performing any actions in your cloud deployment, it is just showing you what Terraform **would do** depending on the current state.

#### Terraform apply and destroy

In order to actually perform the actions described in the `terraform plan` command you have to execute the `terraform apply` command.

> **Warning**: If you are following along this tutorial, after executing `terraform apply` Terraform will create some **actual** cloud resources. This can **cost you money**. <br> <br>
> So, be careful of what you deploy and remember to always **destroy** the resources that you have created after you are done playing around with them. <br>

Run `terraform apply`, Terraform will perform the necessary modifications to get your deployment from the **prior state** to the **current state**.

After a while your resources should be up and running. You can access the AWS web console (in the region that you chose in your `providers.tf` file) in order to see the `EC2 instance` that you just created.

When you are done testing your newly created resources, you can run `terraform destroy` to destroy all the resources created in your deployment.

## What have we learnt?

The code shown in this guide might be very simple, and it does not define all the required resources to create a proper deployment with an `EC2 instance` (for example, the security groups are missing).

But it showed us that it is extremely easy to create and destroy cloud resources with Terraform. If we are very frequently deploying an EC2 instance with a web service, as I previously described in the introduction of this article, it might be **very beneficial** to migrate our deployment to Terraform.

In doing that we would be avoiding the cumbersome process of manually creating an EC2 instance in the AWS web console.
Furthermore, we could even integrate our Terraform files into a CD pipeline, so that a deployment is automatically performed each time we commit to a branch in one of our repositories.

[@[Eduardo](https://github.com/erodrigufer)]

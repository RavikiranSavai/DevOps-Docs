## DevOps Docs.
![PoorasInfoTech](https://poorasinfotechlogo.s3.amazonaws.com/My-DP.jpg)
                                                      Terraform Version >0.12
---

## Table of contents:

* [Docker](#Docker)

* [Kubernetes](#Kubernetes)

* [AWS](#AWS)

* [Ansible](#Ansible)

* [Chef](#Chef)

* [Jenkins](#Jenkins)

* [Jfrog](#Jfrog)

* [linux](#linux)

* [Maven](#Maven)

* [Nexus](#Nexus)

* [Nginx](#Nginx)

* [Selenium](#Selenium)

* [ShellScript](#ShellScript)

* [SonarQube](#SonarQube)

* [Terraform](#Terraform)

* [Tomcat](#Tomcat)

---

## What is this?

---

Here you can find the Terraform configuration for Annalect Infrastructure on AWS Cloud.

## What is Terraform?

---

[Terraform](https://www.terraform.io/) is an infrastructure-as-code tool that greatly reduces the amount of time needed to implement and scale our infrastructure. It's provider agnostic so it works great for our use case.

## Terraform Best Practices

---

Here are Terraform common recommendations that we choose to follow for structuring infrastructure code at Annalect.

  1. Separate Terraform modules and stacks (root module and nested module
  2. Remote backend s3 (encrypted) and DynamoDB to handle the concurrency locking and Terraform state files
  3. Build independent project components in multiple layers (account/region/environment as well as networking/computing/etc)
  4. Pin all modules and providers to a specific version or tag
  5. Use [Snake Case](https://en.wikipedia.org/wiki/Snake_case) for all resource names
  6. Start your project using remote backend to store state
  7. Follow a consistent structure and naming convention
  8. Don’t write the same code again (reusability)
  9. Keep environment configuration separate to maintain it easily
  10. Don't hardcode values which can be passed as variables or discovered using data sources
  11. Use data sources and terraform_remote_state specifically as a glue between infrastructure modules within composition.
  12. Always use the plan command and scan the output carefully and check if Terraform plans to delete a resources that you probably don’t want deleted.
  13. If you do want to replace a resource, think carefully about whether its replacement should be created before you delete the original. If so, you might be able to use create_before_destroy to make that happen.
  14. If you want to change or rename the identifier associated with a resource without accidentally deleting and recreating that resource, you’ll need to update the Terraform state accordingly. You should never update Terraform state file by hand, instead use the terraform state mv command to do it for you.
  15. Avoid hard coding the resources
  16. Validate and format terraform code before committing to repo
  17. Beware of accidental duplicates
  18. Use separate state files for each environment, layer and component. This is about limiting your blast radius.
  19. Turn on debug when you need to do troubleshooting

## How the IaC repository is structured

---

When maintaining infrastructure through Terraform, it’s recommended that a two-repo structure is used. We separated our Terraform code into follwoing two repositories -

**root module**

The [root module ]() holds the Terraform configuration files that are applied to build our desired infrastructure, these provide an entry point into any nested modules we might utilize.

**child module**

The child or [ nested module](https://gitlab.com/) is where the blueprints of the infrastructure are stored. This is the shared repo where Infrastructure engineering and DevOps teams would contribute their infrastructure definitions or modules.

At Annalect, our approach is to build TF upon the existing best practices (e.g. ‘Terraservice’), by creating multiple layers within a code in order to isolate Terraform state via file layout.

Each account has its own directory in terraform_live repo and each account has multiple folder directorires representing different layers under a region and environment.

Here're the layers:

**bootstrap layer**

 This layer take care of all the prerequisites for the new account including following -

- configure S3 bucket and DynamoDB table that will later be used for a remote state backend.

- adding initial IAM roles and policies for infrastructure/devops team

- configure S3 bucket and DynamoDB table that will later be used for a remote state backend.

We maybe even lock down some security parts, like enabling CloudTrail,  setting up AWS Config at account level.

**Global layer**

  A place to put global resources which are account-specific, and things to be created once and not multiple times such as Route53. We prefer using this layer for resouce like S3 or Athena which are not neither application-specific nor environment-specific though these are tied to specific region.

**Networking layer**

  This layer lives inside each region. This is one of the foundational layers of the infrastructure. We put this layer separate from environment layer.

  It contains all of the networking components allowing the Network engineers to define VPCs, define subnets, create peering links, VPN gateways, etc.,within the VPC in a region.

**Security layer**

  This layer contains all of the introduced controls for an environment for all of the Security Groups and NACLs, allowing security and compliance engineers to define controls without conflicting with changes from the network team.

**Environments**

  The last layer is for multiple **environments** under each region that we use to create resourece in.

  We can think of this layer as everything that actually provide any useful features, business value, or anything needed for our customers or clients.

  The exact environments differ for every project, but the typical ones are as follows:

  * dev:      An environment for development workloads
  * qa:       An environment for qa/testing workloads
  * stg:      An environment for pre-production workloads
  * prod:    An environment for production workloads (i.e., user-facing apps)
  * mgmt: An environment for DevOps tooling (e.g., bastion host, OpenVPN, Bamboo, cron jobs, backup, storage, monitoring, etc)

  Within each environment, there are separate sub-folders for each "component".  The components differ for every project, but here are the typical ones:

  * **compute-tier**

    This is especially to build compute resouces like EC2 instances, autoscaling, ECS or Kubernetes clusters to launch in.

  * **edge-tier**

    The edge topology for this environment to build shared resouces like ALB, ELB, NLB, CloudFront etc

  * **data-tier**

    The data stores to run in this environment, such as MySQL, PostgreSQL on RDS and Aurora or MSSQL or Redshift or DynamoDB. Each data-store could even reside in its own dedicated project (folder) to isolate it from all other data stores.

  * **services**

    The apps or microservices to run in this environment, such as a Python Flask frontend or a API backend. Each app could even live in its own folder to isolate it from all the other apps.

  Under each layer, within each folder/component, there are the actual Terraform configuration files. These are organized according to the following naming convention:

    1. main.tf - locals, data block, terraform block for backend, etc.
    2. providers.tf - specify provider and aliases (managed my pylect-infra-terraform)
    3. variables.tf - input variable declarations
    4. outputs.tf - output variable declarations
    5. versions.tf - pin terraform and provider's version
    6. backend.hcl - contain Terraform backend specification (managed my pylect-infra-terraform)
    7. terraform.tfvars – default common variables (managed my pylect-infra-terraform)
    8. common_tags.tf - Tag resouces as per OMC strategy, update TAG as necessary
    8. project specific *.tf - holds all your module blocks and any needed resources not contained within your nested modules. Recommended to name such file with the type of infrastructure being defined. It is also recommended to keep one file per type of resource being set up (rather than one big file combining everything).

  Notes: 

    1. Our `pylect-infra-terraform` wrapper script eases the creation of all the terraform files other than resouces specific `.tf` files. 
    2. You can update main.tf, variables.tf, outputs.tf, versions.tf as necessary for your project. 
    3. You don't need to touch backend.hcl, providers.tf, terraform.tfvars and common_tags.tf. 
    4. We recommend not to use main.tf for any resouce or module block other than locals and data blocks.


### Directory and File Layout

---

The following illustrates a our Terraform live repository structure with all of the concepts outlined above:

```
.
terraform_live
├── README.md
├── docs
│   ├── BeginnersGuide.md
│   ├── Terraform-Standards.md
│   └── images
│       ├── gitlab-pipeline-for-terraform.png
│       └── terraform-ci-cd-pipeline.png
├── azure
│   └── README.md
├── gcp
│   └── README.md
├── aws
│   ├── accounts
│   │   └── ann01-tioprod|ann02-nonprod-qads|ann03-nonprod-assets|ann04-sandbox|ann05-nonprod-aadl
│   │       ├── aue1|aew1 (region)
│   │       │   ├── dev|qa|stg|prod (environment)
│   │       │   │   ├── dev|qa|stg|prod.tfvars
│   │       │   │   ├── compute-tier
│   │       │   │   │   ├── asg
│   │       │   │   │   ├── ec2
│   │       │   │   │   ├── ecs
│   │       │   │   │   ├── workspaces
│   │       │   │   │   └── eks_fargate
│   │       │   │   ├── data-tier
│   │       │   │   │   ├── aurora_mysql
│   │       │   │   │   ├── aurora_postgresql
│   │       │   │   │   ├── dynamodb
│   │       │   │   │   ├── rds_mssql
│   │       │   │   │   ├── rds_mysql
│   │       │   │   │   └── rds_postgresql
│   │       │   │   ├── edge-tier
│   │       │   │   │   ├── clb
│   │       │   │   │   ├── alb
│   │       │   │   │   ├── nlb
│   │       │   │   │   └── cloudfront
│   │       │   │   └── services (applications)
│   │       │   │   │   ├── audience
│   │       │   │   │   ├── omni
│   │       │   │   │   └── civ2
│   │       │   ├── mgmt
│   │       │   │   ├── backup
│   │       │   │   │   ├── ami
│   │       │   │   │   ├── dlm
│   │       │   │   │   ├── rds_snapshot
│   │       │   │   │   ├── redshift_snapshot
│   │       │   │   │   ├── s3_replication
│   │       │   │   │   └── scheduled_events
│   │       │   │   ├── monitoring
│   │       │   │   │   ├── cloudwatch
│   │       │   │   │   ├── datadog
│   │       │   │   │   ├── rds_snapshot
│   │       │   │   │   ├── redshift_snapshot
│   │       │   │   │   ├── s3_replication
│   │       │   │   │   └── scheduled_events
│   │       │   │   ├── logging
│   │       │   │   │   ├── cloudwatchlogs
│   │       │   │   │   ├── loggly
│   │       │   │   │   └── elastic_search
│   │       │   │   ├── storage
│   │       │   │   │   ├── aws_backup
│   │       │   │   │   ├── efs
│   │       │   │   │   ├── fsx
│   │       │   │   │   ├── s3
│   │       │   │   │   ├── glacier
│   │       │   │   │   └── glacier_deep_archive
│   │       │   │   ├── compute-tier
│   │       │   │   ├── data-tier
│   │       │   │   ├── edge-tier
│   │       │   │   └── services
│   │       │   │       ├── bamboo
│   │       │   │       ├── bastion_host
│   │       │   │       ├── gitlab
│   │       │   │       ├── jenkins
│   │       │   │       └── scheduled_jobs
│   │       │   └── networking
│   │       │       └── base
│   │       ├── bootstrap
│   │       │   ├── terraform-backend
│   │       │   └── infra-admin-roles
│   │       ├── security
│   │       │   ├── secure-baseline
│   │       │   ├── secrets_manager
│   │       │   ├── kms
│   │       │   ├── acm
│   │       │   ├── audit
│   │       │   ├── aws_config
│   │       │   ├── cloudtrail
│   │       │   ├── key_pair
│   │       │   ├── cognito
│   │       │   ├── directory_service
│   │       │   ├── inspector
│   │       │   ├── iam
│   │       │   ├── security_groups
│   │       │   ├── subscription
│   │       │   ├── support
│   │       │   ├── waf
│   │       │   ├── waf_shield
│   │       │   ├── okta
│   │       │   └── guardduty
│   │       ├── global
│   │       │   ├── cloudwatch
│   │       │   ├── cloudwatchloggroup
│   │       │   ├── iam
│   │       │   ├── route53
│   │       │   ├── athena
│   │       │   │       ├── aue1
│   │       │   │       └── aew1
│   │       │   ├── s3aue1
│   │       │   │   ├── agency
│   │       │   │   │   ├── annalect
│   │       │   │   │   │   ├── png
│   │       │   │   │   │   └── att
│   │       │   │   │   ├── heartsscience
│   │       │   │   │   ├── bbdo
│   │       │   │   │   ├── omd
│   │       │   │   │   ├── omg
│   │       │   │   │   ├── phd
│   │       │   │   │   ├── tbwa
│   │       │   │   │   └── resolutionmedia
│   │       │   │   ├── application
│   │       │   │   │   ├── omni
│   │       │   │   │   ├── inspiration
│   │       │   │   │   └── audience
│   │       │   │   ├── team
│   │       │   │       ├── datateam
│   │       │   │       ├── datascience
│   │       │   │       └── marketingscience
│   │       │   ├── s3aew1
│   │       │   ├── sns
│   │       │   ├── sqs
│   │       │   └── iam
│   │       ├── files
│   │       ├── lambda_sources
│   │       ├── scripts
│   │       ├── templates
│   │       └── user_data
│   └── modules -> ../../terraform-aws-modules/modules (symlink to module repo that resides on separate repo)
├── packer
└── INSTALL.md
```
(...)


## Terraform Configurations

---

#### providers.tf

* Annalect standard `providers.tf` to use across any layers in AWS N. Virginia and EU (Ireland) region.

* Keep the required Terraform and provider versions along with providers' default and aliased configuration for providers

* Root modules should use a "~>" constraint to set both a lower and upper bound on versions for the provider

* Re-usable modules should constrain only the minimum allowed version, such as >= 1.0.0.

* Alias configuration in `provider.tf`  is mapped to to the CDS abbrevation code for the respective region

* No modification to providers.tf in any layer within US and EU region is required unless otherwise agreed


Example:

```
# default provider configuration
provider "aws" {
  region              = var.region
  allowed_account_ids = [var.trusted_account["id"]]
  profile             = var.trusted_account["assume_role_profile"]
}

# A non-default, aliased provider configuration for an account in another region
provider "aws" {
  alias               = "aue1"
  region              = var.location_region["aue1"]
  allowed_account_ids = [var.trusting_account["id"]]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
}

# A non-default, aliased provider configuration for an account in another region
provider "aws" {
  alias               = "aew1"
  region              = var.location_region["aew1"]
  allowed_account_ids = [var.trusting_account["id"]]
  assume_role {
    role_arn = var.trusting_account["assume_role_arn"]
  }
}

```

---


#### variables.tf

* Constants related to the component.
* Contains definitions for parameters we either want to keep out of the source code repository
* Use empty variables for all secrets if necessary
* This file must include the following code block at the beginning or end of the file.
* Specify a description for each custom variable

Example:

```
variable "company_name_prefix" {}
variable "trusted_account" {}
variable "trusting_account" {}
variable "location_region" {}
variable "common_tags" {}

variable "region" {
  description = "Region where resources should be created"
  default     = "us-east-1"
}

variable "myvar" {
  description = "My custom variable"
  default     = "my-default"
}


```

#### main.tf

* Though `main.tf` is the primary entry point for declaring resources, we prefer using main.tf for the terraform blocks, declaring locals, data sources and other external dependencies.
* This file must include the following code block at the top of the file. Other variables can be added to this block.


```
#remote backend
terraform {
  backend "s3" {}
}

# Data sources
data "aws_caller_identity" "trusted" {}

data "aws_caller_identity" "trusting" {
  provider = aws.aue1
}

# Locals
locals {
  # Common tags to be assigned
  standard_tags = {
    Environment = var.common_tags["environment"]
    Project     = var.common_tags["project"]
    Authorizers = var.common_tags["authorizers"]
    Client      = var.common_tags["client"]
    Location    = upper(var.common_tags["location"])
    ManagedBy   = "terraform"
  }
}
```

#### outputs.tf

* Declare all output variables
* Use description field for all outputs
* Use well formatted, snake case or hyphenated output names
* Resouces attributes via interpolation, we can also use variable interpolation as well to create ouput varaibla that otehr modules can use via output state file.

```
output "public_ip" {
    value = annalect_server.server_name.ipv4_address
}

```
#### versions.tf

* This file should be be used to hold and lock the provider and terraform versions.
* Re-usable modules should constrain only the minimum allowed version, such as >= 0.12.0
* Root modules should use a ~> constraint to set both a lower and upper bound on versions e.g.

```
terraform {
   required_version = "~> 0.12.0"
   required_providers {
     aws      = "~> 2.33"
   }
 }
```

Notes:

1. If individual terraform files are becoming massive, you can break out certain functionality into separate files.
2. You can always create specific Terraform files that contain actual calls and parameterization of centralized Terraform modules
3. Modules should be sourced from git tags.
4. In any resource block that supports tags the following code should be used:

  `tags = merge(local.common_tags, var.tags)`

   This takes the tag values that are in variable.tf and combines them with any values defined in main.tf in the locals block.


###  How we manage Terraform State

---

   * At Annalect, we use S3 with DynamoDB for Terraform​ remote state

   * Our goal is to achieve full isolation between environments and components within an environment of a given infrastructure component.

   * We built a custom module to create the S3/DynamoDB backend to store the Terraform state and lock.

   * Here are some of the characteristics of our terraform backend state:

        1. Separate bucket per account to keep state separately for each account
        2. Isolation via file layout - separate Terraform folders (and therefore separate state files)
        3. Encrypted state data at rest
        4. CRR to store state in another bucket in another region
        5. Separate Terraform State Log Bucket for audit purpose​
        6. Granular access controls using Identity-Based Policies


## Annalect standards for nested modules

---

Annalect's Terraform modules live at [Annalect Terraform Module Repo](https://gitlab.annalect.com/devops/terraform_aws_modules).

All contributors should follow these conventions -

* Use Semantic Versioning. Every release should have a tag in the x.y.z format. Use semantic versions for tagged releases. Tags may be optionally prefixed with a v (e.g. v1.0.1 or 1.0.1).

* Module subdirectories/repositories should follow the [standard module structure](https://www.terraform.io/docs/modules/create.html#standard-module-structure). This makes it easier for others to jump in and contribute.

* External modules should always be [pinned at a version](https://www.terraform.io/docs/modules/sources.html#ref): a git revision or a version number. This practice allows for reliable and repeatable builds. Failing to pin module versions may cause a module to be updated between builds by breaking the build without any obvious changes in our code. Even worse, failing to pin our module versions might cause a plan to be generated with changes we did not anticipate.

* Consider writing tests and examples, and shipping them in directories of the same name.

* The files in example can be named anything as long as they have .tf as the extension.

* Annalect modules should be sourced from a [Annalect Terraform Module Repo](https://bitbucket.org/annalect/terraform-aws-modules) **specifying the version/tag**.

* DevOps or Infra Engineering team can create reusable and production-grade modules and build a library of modules. All other teams will be able to reuse those modules to deploy common infrastructure.

* Terraform modules should have a README that explains what the module does, why it exists, how to use it, and how to modify it.


Here’s what our project layout for modules might look like:

```
.
├── README.md
├── examples
├── modules
│   ├── README.md
│   ├── tfstate-backend
│   ├── apigateway
│   ├── app_tier
│   ├── batch
│   ├── cloudwatch
│   ├── cloudwatchevent
│   ├── cloudwatchloggroup
│   ├── compute
│   │   ├── asg
│   │   ├── ec2
│   │   ├── ecs
│   │   └── k8s
│   ├── data_tier
│   │   ├── auroa-mysql
│   │   ├── aurora-psql
│   │   ├── dynamodb
│   │   ├── mssql
│   │   └── redshift
│   ├── edge_tier
│   │   ├── alb
│   │   ├── cloudfront
│   │   ├── elb
│   │   └── nlb
│   ├── iam
│   │   ├── account
│   │   ├── assumable-role
│   │   ├── assumable-roles
│   │   ├── assumable-roles-with-saml
│   │   ├── group-with-assumable-roles-policy
│   │   ├── group-with-policies
│   │   ├── iam-account
│   │   ├── iam-assumable-role
│   │   ├── iam-assumable-roles
│   │   ├── iam-assumable-roles-with-saml
│   │   ├── iam-group-with-assumable-roles-policy
│   │   ├── iam-group-with-policies
│   │   ├── iam-policy
│   │   ├── iam-user
│   │   ├── policy
│   │   └── user
│   ├── lambda

```
(..)

We recommend you break down the Terraform module into 3 files:

1. variables.tf contains all the inputs to the module, with defaults if possible. All the information from variables.tf including descriptions will be included in the generated documentation.
2. outputs.tf contains all the outputs from the module. Again, all the information will be included in the documentation.
3. main.tf conatins all the logic and other “code” that implements the module.

Re-usable modules should constrain only the minimum allowed version, such as >= 0.12.0. This specifies the earliest version that the module is compatible with while leaving the user of the module flexibility to upgrade to newer versions of Terraform without altering the module.

Each of the modules will be controlled by DevOps and Infrastructure engineering team, and everyone can contribute to this repository.

When a new module is added to the repository it creates a Pull Request. This Pull Request is reviewed and then merged to the repository to maintain quality control of the modules.

* [Read More about Modules](https://www.terraform.io/docs/registry/modules/publish.html)


### Deployment Strategies

---

Deploying infrastructure code changes is more complicated than deploying application code.

Terraform does not roll back automatically in case of errors.

**Deployment Server**:

  * All of infrastructure code changes should be applied from a CI server and not from a DevOps or developer’s computer.

  * Running Terraform from CI server forces to fully automate the deployment process, ensuring the deployment always happens from a consistent environment.

  * CI server should not be exposed on the public internet and it should be run in private subnets, without any public IP.

  * Lock the CI Server down and make it accessible over HTTPS with 2FA.

  * Consider not giving the CI server permanent admin credential. Instead let CI assume IAM role.



### A Workflow for Deploying Infrastructure as Code

---

  **IMPORTANT**:

  1. Once we adopt Terraform, changes AWS infrastructure should only be managed through this specialized Terraform workflow (not through the AWS Console directly).
  2. Changes in the AWS Console can conflict with Terraform management, resulting in downtime, data loss, or delays to reconcile these manual changes.
  3. It is important that all changes to environment are managed with Terraform.

### CI/CD Pipeline Workflow for Terraform Using GitLab

---

![CI/CD using Terraform and Gitlab](./docs/images/terraform-ci-cd-pipeline.png)

1. The DevOps or Infrastructure Engineer creates feature branch ** from master branch **
    1. creates a folder for the terraform project (if new project) in the appropriate subfolder (account/region/environment)
    2. run start-project script
    3. changes  the Terraform configuration file in his/her local machine and commits the code to feature branch in Gitlab and then create a pull request to merge it into master.

2. Gitlab triggers a continuous integration job.
 
3. Gitlab will pull the latest code from the configured repo which contains Terraform files to its workspace.
 
4. GitLab generates a configuration file for the deployment and then initializes the remote s3 backend & DynamoDBTable.
 
5. Terraform generates a plan about the changes that have to be applied on the infrastructure.
 
6. Gitlab sends a notification to a Teams channel and email about the changes for manual approval. ** And waits for user response **
 
7. Approver can approve or disapprove the Terraform plan.
 
8. The user input is sent to Gitlab for proceeding with the further action.
 
9. Once the changes are approved by an Approver, Gitlab will execute the terraform apply command to reflect the changes to the infrastructure.
 
10. (Terraform's output upon `apply` is logged) --Terraform will create a report about the resources and its dependency created while executing the plan.
 
11. Terraform will provision the resources as approved --in the provided environment.
 
12. Gitlab will again send a notification to the Teams channel and email with the output of the `apply` command -- about the status of the infrastructure after the applying changes on it.

### GenericWorkflow

The workflow is described in the following picture:

![alt text](./docs/images/gitlab-workflow.png)

1. Pull the latest code from the master branch.

2. Create feature branch from the master branch.

3. The devops/contributor works on implementing a solution.

4. Rebase from latest master branch (local-rebase with master).
    
    * Note: If you don't want to trigger CI/CD Pipeline on commit add **[CI-SKIP]**(followed by your message) in the commit message, this will not trigger Pipeline(Only if your work is still in WIP).


5. The devops/contributor pushes the working branch onto the centralized repository (CI/CD Pipeline will be triggered - Validate-**Review** (this will look for terraform fmt,terraform docs and check if there is any merge conflict) stage (detect-and-process-folder-change.sh). Email Notification will be sent to user with plan details on success, if not again user needs to work until pipeline is successfull) (Also send notification only if pipeline is failed).


6. The devops/contributor proposes a merge-request/pull-request(Email/Teams(Gitlab Channel) notification will be sent to MR approver )(detect-and-process-folder-change.sh).

    * **IMPORTANT**: Make sure you do the  **rebase** from the latest master branch and check if there is any conflict, before sending merge request.

    * By opening a MR/PR, the CI/CD Pipeline is triggered automatically (**Onmergerequest** - Stage).

    * CI/CD job will run on the proposed merge commit onto master branch.

    * CI/CD job will notify in the MR/PR comments the job status.
      * In case of FAILURE, will ask the devops/contributor to fix the issue
      * MR/PR should be automatically updated and CI job retriggered.
      
7. Once the CI/CD job status is OK, the Approver may perform manual inspections.

    * The maintainer (or CI/CD admin) assign one or several Approver(s) to review the code.
    * Reviewer(s) will post the form with checked items as a commment in the merge-request webpage
    * Reviewer(s) may start a discussion with contributor(s) based on the form results.

8. The Approver approves the MR/PR. 

    * The CI/CD is triggered automatically.
    * CI/CD job will notify build status in the master. 
        * FAILURE should not occur very often.
        * In case of FAILURE, devops/integrator may go back onto working branch or directly in develop branch (steps 3, 4, 5).
        * Once the CI/CD passed, merge request is accepted and merged.
        * It will automatically closed the approved MR/PR(s).

9. Since CI/CD job was OK on the real merged master branch, the approver:

    * Close the MR/PR (if it is not automatically done when merged).

10. If CI/CD is failed, again devops/contributor need to re-work.

11. The approver SHALL delete the branch (if not automatically done when merged).

### Automatic Terraform documentation with terraform-docs
---

As part of our automated continuous integration pipeline, we’ve adapted `terraform-docs` to keep Terraform module's documentation updated.

In its simplest invocation with pre-commit Git hook, it adds the generated documentation to the README for all of the variables and outputs along with the descriptions.


### Golden Rule of Terraform

---

* After you start using Terraform, you should only use Terraform (not use AWS Console or CLI for any changes on the infrastructure).

* When a part of the infrastructure is managed by Terraform, we should never manually make changes to it via a web UI, or manual API calls, or any other mechanisms. Changes in the AWS Console/CLI can conflict with Terraform management, resulting in downtime, data loss, or delays to reconcile these manual changes.

* The only way to ensure that the Terraform code in the live repository is an up-to-date representation of what’s actually deployed is to never make out-of-band changes. Out-of-band changes not only lead to complicated bugs, but they also void many of the benefits we get from using IaC in the first place.

* Always deploy from a single **master** branch as a single source of truth for Terraform

* We should only have to look at a single branch to understand what’s actually deployed. That means all changes that affect the environment should go directly into master (you can create a separate branch, but only to create a pull request with the intention of merging that branch into master) and `terraform apply` should run against the master branch only through CI/CD pipeline.


### Team Structure to manage Terraform code

---

We use a per-layer based access controls to delegate ownership of components. It’s very important to maintain Terraform code integrity, quality and also to ensure security and compliance of the platform.

**Who's responsible for which infrastructure layer -**
```
|---------------------------|--------------------------------------------------------------------|
|  **Layers**               |         **Role**                                                   |
|---------------------------|--------------------------------------------------------------------|
| bootstrap                 |         DevOps - Admin                                             |
|---------------------------|--------------------------------------------------------------------|
| security                  |         DevOps - Security                                          |
|---------------------------|--------------------------------------------------------------------|
| networking                |         DevOps - Network Admins                                    |
|---------------------------|--------------------------------------------------------------------|
| global                    |         DevOps                                                     |
|---------------------------|--------------------------------------------------------------------|
| data_tier                 |         DevOps - DBA                                               |
|---------------------------|--------------------------------------------------------------------|
| edge_tier                 |         DevOps                                                     |
|---------------------------|--------------------------------------------------------------------|
| services                  |         DevOps,  component/application/development team in DEV env |
|---------------------------|--------------------------------------------------------------------|
| monitoring                |         DevOps                                                     |
|---------------------------|--------------------------------------------------------------------|
| logging                   |         DevOps                                                     |
|---------------------------|--------------------------------------------------------------------|
| backup                    |         DevOps                                                     |
|---------------------------|--------------------------------------------------------------------|
```

  Notes:

  * We believe that this separation removes risk and complexity.
  * The respective component/application team could be contributors to respective `service` layer. However, they do not have approval to apply changes directly.
  * The Infrastructure team can vet and review infrastructure as code configuration files written by component/application team.

### Adapting IaC in Team to Be Successful
---

* Since our team is used to managing infrastructure manually and making all the changes directly, switching to infrastructure as code required more than just introducing a new tool or technology.

* If we want to successfully adopt IaC, it is also requires changing the culture and process of the team.

* Making changes with Terraform initially takes time, but we’ll have to invest time learning Terraform with all available resources

* You can quickly get up to speed by reading the [documentation](https://www.terraform.io/docs/index.html) and [guides](https://www.terraform.io/guides/index.html) that HashiCorp provides for Terraform

* [Learn about provisioning infrastructure with Terraform](https://learn.hashicorp.com/terraform)

---

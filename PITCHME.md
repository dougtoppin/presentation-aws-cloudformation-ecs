## Using AWS CloudFormation to Create and Manage ECS Clusters

AWS DC Meetup

01-May-2018

dougtoppin@gmail.com

[github.com/dougtoppin/presentation-aws-cloudformation-ecs](https://github.com/dougtoppin/presentation-aws-cloudformation-ecs)

---

## Agenda

- What is CloudFormation
- Why use it
- Where did it help me
- Nested stacks
- ECS cluster
- Examples
- Lessons learned
- Links

---
### What is CloudFormation

"AWS CloudFormation provides a common language for you to describe and provision all the infrastructure resources in your cloud environment. CloudFormation allows you to use a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This file serves as the single source of truth for your cloud environment. "

---
### Why use CloudFormation

* Staying within the AWS ecosystem has advantages

* IAM
* logging
* cli
* SDK
* admin console

+++

* No other tools to learn, install, configure, support and pay for

+++

* it is comprehensive
* package all of your resources and creation/configuration/deletion orchestration into one or more templates
* outside tools can be installed or configuration performed as a part of the template

+++

If you have a demo or similar application to run in AWS and your instructions have the user manually create resources (security groups for example) and manually launch an EC2 instance you are going to lose some opportunities.

Instead, create a CloudFormation template in a public bucket and have users launch it

+++

Using it with existing templates requires less knowledge of AWS resource details

---

### Where did it help me

* My datacenter (ECS cluster, ALB, RDS, applications) is a yaml file
* Create an entire environment with a single command, again and again
* Don't pay for idle, delete an environment when you are not using it

+++

Easily and reliably create dev, qa, CICD and production environments in parallel using the same templates

+++

Namespace based environment (stack names) means that multiple environments can coexist without stepping on each other

+++

Ease of environment creation facilitates things like automated regression testing such as a Jenkins pipeline I&T stage that spins up everything and then deletes it

---
### Nested Stacks

* Group related/hierarchical resources into templates
* Templates can invoke/reference other templates

---

### ECS cluster

[github.com/awslabs/ecs-refarch-cloudformation](https://github.com/awslabs/ecs-refarch-cloudformation)

* Excellent example of nested and application stacks
* ECS EC2 based cluster
* master, VPC, security groups, ALB, ECS instances, products service, website service
* Configures SSM Run Command instead of a bastion host, cheaper and improved security

+++

ECS template set great starting point to build upon such as

* autoscaling group policy and CloudWatch metric with alarm on high cluster memory reservation to invoke it
* RDS stacks
* application stacks

+++

Application stacks can be created and deleted independently

+++

Stack layers being created

![complete stack](assets/aws-stacks-in-progress-01.png)

+++

Where this stack is in the resource creation process

![complete stack](assets/aws-events-01.png)

+++

A stack can provide consumable outputs

![complete stack](assets/aws-outputs-01.png)

+++

What resources this stack created

![complete stack](assets/aws-resources-01.png)

+++

Good example of reusable stack layers

![complete stack](assets/aws-stacks-01.png)

+++

Don't forget necessary roles for automation tools such as Lambda functions

![complete stack](assets/aws-role-01.png)

---

### Lessons learned

* Move quickly from manual creation to automation
* Don't get in the habit of creating things using the admin console or even the cli without scripting it, it is difficult to recreate exactly, document and support things created using the admin console

+++

* Even better, script in Lambda which makes moving to Step Functions easier than if you use /bin/sh

+++

* Sometimes a stack delete will fail (race condition for example), you might need to try it again or do a manual delete of a resource, do not assume that a stack delete will work every time without confirming

+++

* Don't hardcode stuff in your templates
* Use the SSM Parameter Store to centralize settings
* Pass parameters between templates

+++

* Use `wait` if you need to ensure that resource creation/deletion is orchestrated before continuing to the next action

+++

* When creating resources with names/keys prefix the environment name to prevent conflicts if there is any chance that multiple instances of a stack might exist

+++

* CloudFormation does not support SecureStrings in the Parameter Store but it is in the AWS queue to get working someday

+++

* Start with an existing set of templates rather than creating your own from scratch

+++

* stack-updates seem to take a while vs delete/create cycles

+++

* To prevent race conditions between stacks use CloudFormation ouputs/export and ImportValue, this will cause a dependency/wait to occur. This also helps in creating a stack that needs something to exist but the "base" stack was never created.

+++

* Stack creates can take a while to complete, particularly if there are numerous dependencies, use `wait` to allow creations to occur in parallel and save time

+++

* Note that the Parameter Store is rate limited, be careful about having CloudFormation do a lot of put values without some rate governing

+++

* Organize CloudWatch log groups by log message format, then forward desired group(s) to your ElasticSearch cluster

+++

* Instead of using the AWS cli to create stacks use Step Functions and Lambda right from the start, this lets creates/deletes run serverless and asynchronous (no blocked foreground shell)

+++

* Instead of deleting and recreating an application stack you might be able to just force a redeploy of a container which will pull a new image which saves time when testing

```
aws ecs update-service --cluster xxx --service yyy \
    --force-new-deployment
```

+++

* Use Jenkins jobs to automate basic functions by invoking CloudFormation commands

+++

* Note that account limits might cause a create to fail, monitor your limits usage and settings before you have a create fail at an inconvenient time

+++

* Plan your IAM roles/policies so that Lambda automation does not have free reign, this is harder than it sounds

+++

Use ```aws cloudformation validate-template``` and ```cfn_nag``` before you start a long stack creation that might fail at a late stage

+++

* Disable rollback on stack creates when you need to debug stack issues

+++

* Note that the AMI to launch is configured in ecs-cluster.yaml, you might want to keep this updated or even better invoke a Lambda function to populate it in real-time during stack creation

+++

* Make your instance types a parameter and potentially set them differently between dev, test, prod environments

---

## Links

- [github.com/awslabs/ecs-refarch-cloudformation](github.com/awslabs/ecs-refarch-cloudformation)
- [aws.amazon.com/cloudformation/](aws.amazon.com/cloudformation)
- [docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html#cli-aws-cloudformation](docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html#cli-aws-cloudformation)
- [docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [cloudonaut.io/optional-parameter-in-cloudformation/](cloudonaut.io/optional-parameter-in-cloudformation)
- [github.com/stelligent/cfn_nag](https://github.com/stelligent/cfn_nag)

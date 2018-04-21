## Using AWS CloudFormation to Create and Manage ECS Clusters

AWS DC Meetup

01-May-2018

dougtoppin@gmail.com

[github.com/dougtoppin/presentation-aws-cloudformation-ecs](github.com/dougtoppin/presentation-aws-cloudformation-ecs.git)

---

## Agenda

- What is CloudFormation
- Why use it
- Where did it help me
- Nested stacks
- Examples
- Lessons learned
- Links

---
### What is CloudFormation

"AWS CloudFormation provides a common language for you to describe and provision all the infrastructure resources in your cloud environment. CloudFormation allows you to use a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This file serves as the single source of truth for your cloud environment. "

---
### Why use it

Staying within the AWS ecosystem has advantages

* IAM
* logging
* cli
* admin console

+++

No other tools to learn, install, configure, support and pay for

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

My datacenter (ECS cluster, ALB, RDS, applications) is a yaml file

Create an entire environment with a single command, again and again

Don't pay for idle, delete an environment when you are not using it

+++

Easily create dev, qa, CICD and production environments

+++

Namespace based environment means that multiple environments can coexist without stepping on each other

+++

Ease of environment creation facilitates things like automated regression testing such as a Jenkins pipeline I&T stage that spins up everything and then deletes it

+++

Added autoscaling group policy and CloudWatch metric with alarm on high cluster memory reservation to invoke it

---
### Nested Stacks

master, VPC, security groups, ALB, ECS instances

Group related/hierarchical resources into templates

Templates can invoke/reference other templates

---

Create an ECS cluster

[github.com/awslabs/ecs-refarch-cloudformation][github.com/awslabs/ecs-refarch-cloudformation]

+++

Examples of stacks that provide a complete system

* base cluster
* resources such as RDS
* application services
* SNS items

---

### Lessons learned

Move quickly from manual creation to automation

Don't get in the habit of creating things using the admin console, it is difficult to recreate exactly, document and support things created using the admin console

+++

deleting stacks

Sometimes a stack delete will fail (race condition for example), you might need to try it again or do a manual delete of a resource, do not assume that a stack delete will work every time without confirming

+++

Don't hardcode stuff in your templates

* use the SSM Parameter Store to centralize settings
* pass parameters between templates

+++

Use `wait` if you need to ensure that resource creation/deletion is orchestrated before continuing to the next action

+++

When creating resources with names/keys concatenate the environment name to prevent conflicts if there is any chance that multiple instances of a stack might exist

+++

CloudFormation does not support SecureStrings in the Parameter Store but it is in the AWS queue to get working someday

+++

Start with an existing set of templates rather than create your own from scratch

+++

stack-updates seem to take a while vs delete/create cycles

+++

To prevent race conditions between stacks use Ouputs/export and ImportValue, this will cause a dependency/wait to occur. This also helps in creating a stack that needs something to exist but the "base" stack was never created.

+++

Stacks can take a while to complete, particularly if there are numerous dependencies, use `wait` to allow creations to occur in parallel and save time

+++

Note that the Parameter Store is rate limited, be careful about having CloudFormation do a lot of put values without some rate governing

+++

Organize CloudWatch log groups by log message format, then forward desired group(s) to your ElasticSearch cluster

+++

Instead of using the AWS cli to create stacks use Step Functions and Lambda right from the start, this lets creates/deletes run serverless and asynchronous (no blocked foreground shell)

---

## Links

- [aws.amazon.com/cloudformation/](aws.amazon.com/cloudformation)
- [docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html#cli-aws-cloudformation](docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html#cli-aws-cloudformation)
- [docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [cloudonaut.io/optional-parameter-in-cloudformation/](cloudonaut.io/optional-parameter-in-cloudformation)

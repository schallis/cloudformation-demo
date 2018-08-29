Cloudformation Demo
===================

## 1. Creating a basic template
-------------------------------

Let's create a single EC2 instance! To help us, available resources are documented and we can look up properties and values etc. See documentation at https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html.

Take a look at file [1-basic-ec2.yml](1-basic-ec2.yml)

You can spin this up from the command line (on Linux) with the following commands:

    $ aws cloudformation create-stack --stack-name "Demo-Stack-Steve" --template-body file://basic-ec2.yaml
    $ aws cloudformation list-stacks --max-items 5

**Problems:**
- Hard coded type (t2.micro), may want to change later
- Whole bunch of stuff we didn't specify
    - Tags
    - Security Group
    - VPC/Subnet

Let's fix these in our next template...

## 2. Updating the template and stack
----------------------------------

Take a look at file [2-updated-ec2.yml](2-updated-ec2.yml)

In this new template we pull out the Instance Type into a parameter to let the user decide at deploy-time.
We've also added Tags to both the stack and our EC2 Resource.

Note that our EC2 instance will be terminated and recreated when we change the type, but will remain and simply be updated if we just change the name.

    $ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://2-updated-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium --tags Key=Owner,Value=StevenChallis

**Problems**
- Our subnet is hard coded now, how can I deploy this in multiple accounts?
- Hard coded AMI won't work in another region


## 3. Specifying mappings
----------------------

Take a look at file [3-mappings-ec2.yml](3-mappings-ec2.yml)

We've now pulled out subnet and AMI into a mapping so it can change based on the region.
We'll also turn on detailed monitoring only if we're deploying to Production, basic if not.

    $ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://3-mappings-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium,ParameterKey=Environment,ParameterValue=Prod --tags Key=Owner,Value=StevenChallis
    
**Problems:**
Our solution isn't Highly Available. We likely want multiple instances of our application spread across Availability Zones. We might even just want this in Production but not in Development.


## 4. Autoscaling EC2 instances
----------------------------

Let's replace our single instance with an Autoscaling Group of instances whose size depends
on the environment and customer demand. Notice how most of the instance configuration
is tranferred as-is to the Launch Config.

Take a look at file [4-autoscaling-ec2.yml](4-autoscaling-ec2.yml)

    $ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://4-autoscaling-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium,ParameterKey=Environment,ParameterValue=Prod --tags Key=Owner,Value=StevenChallis


## 5. Delete the stack
-------------------

One advantage of a stack is that it makes it easy to remove a collection of resources when
we are done. Let's see this in action by deleting our stack... we know we can easily recreate it!

    $ aws cloudformation delete-stack --stack-name "Demo-Stack"


## 6. Now, let's construct one your workloads
---------------------------------------------

It might be helpful to draw out the components to help determine what we will need in our template


Further Learning
================

See more examples of templates for common scenarios at https://aws.amazon.com/quickstart/

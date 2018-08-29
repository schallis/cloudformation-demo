Cloudformation Demo
===================

1. Creating a basic template
----------------------------

Let's create a single EC2 instance

See documentation at https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
- Show different resources, links and examples

$ aws cloudformation create-stack --stack-name "Demo-Stack-Steve" --template-body file://basic-ec2.yaml
$ aws cloudformation list-stacks --max-items 5

*Problems:*
- Hard coded type (t2.micro), may want to change later
- Whole bunch of stuff we didn't specify
    - Tags
    - Security Group
    - VPC/Subnet

Let's fix these in our next template...


2. Updating the template and stack
----------------------------------

In this new template we've pulled out the Instance Type into a parameter.
We've also added Tags to both the stack and our EC2 Resource.

Note that the instance is terminated when we change the type, but not if we just change the name.

$ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://2-updated-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium --tags Key=Owner,Value=StevenChallis

*Problems*
- Our subnet is hard coded now, how can I deploy this in multiple accounts?
- Hard coded AMI won't work in another region


3. Specifying mappings
----------------------

We've now pulled out subnet and AMI into a mapping so it can change based on the region.
We'll also turn on detailed monitoring if we're deploying to Production, basic if not.

$ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://3-mappings-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium,ParameterKey=Environment,ParameterValue=Prod --tags Key=Owner,Value=StevenChallis


4. Autoscaling EC2 instances
----------------------------

Let's replace our single instance with an autoscaling group whose size depends
on the environment and customer demand. Notice how most of the instance configuration
is tranferred as-is to the Launch Config.

$ aws cloudformation update-stack --stack-name "Demo-Stack-Steve" --template-body file://4-autoscaling-ec2.yml --parameters ParameterKey=InstanceTypeParameter,ParameterValue=t2.medium,ParameterKey=Environment,ParameterValue=Prod --tags Key=Owner,Value=StevenChallis


5. Delete the stack
-------------------

One advantage of a stack is that it makes it easy to remove all resources when
we are done. Let's see this by deleting our stack.

$ aws cloudformation delete-stack --stack-name "Demo-Stack"


6. Let's construct your workload
--------------------------------

We'll draw out the components and determine what we need in our template


Further Learning
================

See more examples of templates at https://aws.amazon.com/quickstart/
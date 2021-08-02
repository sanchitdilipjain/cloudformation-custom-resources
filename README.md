## Cloudformation Custom resources

- What is AWS Cloudformation?

    - AWS CloudFormation offers an easy way to deploy a collection of related AWS and third-party resources, provision them faster, and manage them throughout their lifecycles, by treating infrastructure as code. 

    - A CloudFormation template contains information for the desired resources and their dependencies so we can launch and configure them together as a stack. We can use templates to create, update, and delete an entire stack as a single unit, instead of managing resources individually. We can also manage and provision stacks across multiple AWS accounts and AWS Regions.

    - However, Cloudformation fails at certain places, such as executing a lambda via deploying the stacks or needing to know the CIDR Block for the VPC from a given VPC ID. There we can leverage AWS CloudFormation Custom Resource to overcome these loopholes

- What are CloudFormation Custom Resources?

    - A custom resource is a custom type powered by a Lambda function to execute a task that CloudFormation cannot do, for example, retrieving an AMI ID or the CIDR block for a VPC. 
    
    - Custom resources have a “request type” included with the request, allowing the custom resource to create, update and delete whatever it is doing. Outside to just normal lookup, there are more capabilities an AWS CloudFormation Custom Resource can provide

- How Does the Custom Resource Work?
      
  <p align="center"><img src="docs/images/image1.png" class="inline" width="700" height="400"/></p>
   
- Demo Blog <a href="https://sanchitdilipjain.github.io/aws-layered-vpc-architecture/">link</a>     

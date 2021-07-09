## Cloudformation Custom resources

1. What is AWS Cloudformation?

    - AWS CloudFormation offers an easy way to deploy a collection of related AWS and third-party resources, provision them faster, and manage them throughout their lifecycles, by treating infrastructure as code. 

    - A CloudFormation template contains information for the desired resources and their dependencies so we can launch and configure them together as a stack. We can use templates to create, update, and delete an entire stack as a single unit, instead of managing resources individually. We can also manage and provision stacks across multiple AWS accounts and AWS Regions.

    - However, Cloudformation fails at certain places, such as executing a lambda via deploying the stacks or needing to know the CIDR Block for the VPC from a given VPC ID. There we can leverage AWS CloudFormation Custom Resource to overcome these loopholes

2. What are CloudFormation Custom Resources?

    - A custom resource is a custom type powered by a Lambda function to execute a task that CloudFormation cannot do, for example, retrieving an AMI ID or the CIDR block for a VPC. 
    
    - Custom resources have a “request type” included with the request, allowing the custom resource to create, update and delete whatever it is doing. Outside to just normal lookup, there are more capabilities an AWS CloudFormation Custom Resource can provide

3. How Does the Custom Resource Work?
      
      <img src="images/image1.png" class="inline" width="700" height="400"/>

    - Let's assume we are deploying the below custom resource as part of the CloudFormation template.
    
        ```MyResource:
        Type: Custom::Test
        Properties:
            ServiceToken: LambdaARN
            Name: “Value”
            List: ["1","2","3","4"]
         ```
         
    - When CloudFormation runs the above input, it uses the ServiceToken to lookup the Lambda function it is aspected to invoke and sends the parameters to that function.
        
        ``` {
              "RequestType" : "Create",
              "ResponseURL" : “foo”,
              "StackId" : ”bar”,
              "RequestId" : "unique id”,
              "ResourceType" : "Custom::Test",
              "LogicalResourceId" : "MyResource",
              "ResourceProperties" :
               {
                 "Name" : "Value", 
                 "List" : [ "1", "2", "3" ]
               }
             }
         ``` 

    - The request also contains a pre-signed URL leveraged by the Lambda function to return the response. All of the interaction between the custom resource and CloudFormation is conducted via this pre-signed URL. If the custom resource doesn’t return the information to the pre-signed URL, the CloudFormation will not complete. If we don't specify a timeout for the stack creation, the stack will hang for hours.

        ```{
              "RequestType": "Create", 
              "ServiceToken": LambdaARN, 
              "ResponseURL": "foo",
              "StackId": "bar", 
              "RequestId": "unique id", 
              "LogicalResourceId": "MyResource", 
              "ResourceType’: "Custom::Test", 
              "ResourceProperties":{
                "ServiceToken": LambdaARN, 
                "Name" : "Value", 
                "List" : [ "1", "2", "3" ]
                }
              }
         ```

    - In this sample request, there are several entries:

        - RequestType: It holds either create, update or delete. And used to assist the custom resource to decide what to do

        - ServiceToken: service token holds Custom resource ARN where CloudFormation needs to send the requests

        - ResponseURL: This is the pre-signed URL where the response must be sent back to the CloudFormation.

        - StackId: stack ID contains the custom resource request.

        - RequestId: AWS provided an unique request ID.

        - LogicalResourceId: name of the logical resource as it appears in the stack.

        - ResourceType: name of the resource as mentioned in the stack.

        - ResourceProperties: parameters that are provided to the function.

    - After the custom resource function executes, it returns the following response to CloudFormation using the pre-signed URL provided in the Request data.
     
         ```{
            "Status" : "SUCCESS",
            "PhysicalResourceId" : “baz”,
             "StackId" : “bar”,
             "RequestId" : "unique id”,
             "LogicalResourceId" : "MyResource"
           } 
         ```

    - Next, CloudFormation continues the execution with the rest of the stack deployment operations

4. Demo 

    - In this demo, we will leverage Cloudformation custom resource to copy data from one S3 bucket to other S3 bucket, and post successful copy the cloudformation stack will be completed

    - The following diagram shows the architectural components of the solution:
    
        <img src="images/image2.png" class="inline" width="300" height="300"/>

    - <a href="https://github.com/sanchitdilipjain/cloudformation-custom-resources/blob/main/custom-resource-template.yaml">Link to Cloudformation template</a>
    
    - Steps to deploy Cloudformation
    
        1. Open the AWS CloudFormation console at<a href="https://console.aws.amazon.com/cloudformation/"> link </a>

        2. Choose Create Stack.

        3. In the Template section, choose to Upload a template file, and then choose a file under the upload section
            
            <img src="images/image3.png" class="inline" width="700" height="400"/>

        4. Choose Next.

        5. In the Stack name field, type custom-resource-tutorial, and under the Parameters section, provide the following parameter, and then choose Next.
        
            - DestinationBucket
            
            - SourceBucketName
            
            - SourceObjects
            
            - SourceS3KeyPrefix
            
            <img src="images/image4.png" class="inline" width="700" height="400"/>

        6. For this walkthrough, we will preserve all advanced settings as a default state, so choose Next.

        7. Ensure that the stack name and template URL are correct, and then choose Create to.
        
            <img src="images/image5.png" class="inline" width="700" height="400"/>
            
            <img src="images/image6.png" class="inline" width="700" height="200"/>
       
        8. If the custom resource is executed successfully then the Cloudformation stack will be in Create Complete status, if due to some reason the custom resources execution failed then Cloudformation will be stuck for hours if the timeout is not mentioned or will be in Failed status once the timeout is over.
        
            - Cloudformation in-progress state
            
               <img src="images/image7.png" class="inline" width="700" height="250"/>
            
            - Successful State
               
               <img src="images/image8.png" class="inline" width="700" height="400"/>
            
            - Failed State
              
               <img src="images/image9.png" class="inline" width="700" height="200"/>
     
        
        

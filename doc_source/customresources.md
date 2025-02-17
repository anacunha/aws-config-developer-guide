# Record Configurations for Third\-Party Resources<a name="customresources"></a>

Record configurations for third\-party resources or custom resource types such as on premise servers, SAAS monitoring tools, and version control systems \(like GitHub\)\. You can publish the configuration data of third\-party resources into AWS Config and view and monitor the resource inventory and configuration history using AWS Config console and APIs\. Now, you can use AWS Config to manage all your resources and evaluate resource configuration for compliance against best practices using AWS Config rules\. You can also create AWS Config rules or conformance packs to evaluate these third\-party resources against best practices, internal policies, and regulatory policies\. 

**Note**  
If you have configured AWS Config to record all resource types, then third\-party resources that are managed \(created, updated, or deleted\) through AWS CloudFormation are automatically tracked in AWS Config as configuration items\.

**Prerequisite:** The third\-party resources or custom resource type must be registered using AWS CloudFormation\. 

**Topics**
+ [Step 1: Setup Your Development Environment](#customresources-setupdevenvironment)
+ [Step 2: Model Your Resource](#customresources-modelresource)
+ [Step 3: Generate Artifacts](#generateartifactsforcfnsubmit)
+ [Step 4: Register Your Resource](#customresources-registerresource)
+ [Step 5: Publish Resource Configuration](#customresources-publishresourceconfiguration)
+ [Record and Delete a Configuration State for Third\-Party Resources Using AWS CLI](customresources-cli.md)
+ [Managing a Configuration State for Third\-Party Resources Type Using APIs](customresources-api.md)
+ [Region Support](#custom-resources-regions)

## Step 1: Setup Your Development Environment<a name="customresources-setupdevenvironment"></a>

Install and configure AWS CloudFormation AWS CLI\. The AWS CLI allows you to model and register your custom resources\. For more information, see [Custom Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html) and [What Is the CloudFormation Command Line Interface?](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/what-is-cloudformation-cli.html)\.

## Step 2: Model Your Resource<a name="customresources-modelresource"></a>

Create a resource provider schema that conforms to and validates the configuration of the resource type\.

1. Use the `init` command to create your resource provider project and generate the files it requires\.

   ```
   $ cfn init
   Initializing new project
   ```

1. The `init` command launches a wizard that walks you through setting up the project, including specifying the resource name\. For this walkthrough, specify `MyCustomNamespace::Testing::WordPress`\.

   ```
   Enter resource type identifier (Organization::Service::Resource): MyCustomNamespace::Testing::WordPress
   ```

1. Enter a package name for your resource\.

   ```
   Enter a package name (empty for default 'com.custom.testing.wordpress'): com.custom.testing.wordpress 
   Initialized a new project in /workplace/user/custom-testing-wordpress
   ```
**Note**  
In order to guarantee that any project dependencies are correctly resolved, you can import the generated project into your IDE with Maven support\.  
For example, if you are using IntelliJ IDEA, you would need to do the following:  
From the **File** menu, choose **New**, then choose **Project From Existing Sources**\.
Navigate to the project directory
In the **Import Project** dialog box, choose **Import project from external model** and then choose **Maven**\.
Choose **Next** and accept any defaults to complete importing the project\.

1. Open the `mycustomnamespace-testing-wordpress.json` file that contains the schema for your resource\. Copy and paste the following schema into `mycustomnamespace-testing-wordpress.json`\.

   ```
   {
     "typeName": "MyCustomNamespace::Testing::WordPress",
     "description": "An example resource that creates a website based on WordPress 5.2.2.",
     "properties": {
       "Name": {
         "description": "A name associated with the website.",
         "type": "string",
         "pattern": "^[a-zA-Z0-9]{1,219}\\Z",
         "minLength": 1, "maxLength": 219
       },
       "SubnetId": {
         "description": "A subnet in which to host the website.",
         "pattern": "^(subnet-[a-f0-9]{13})|(subnet-[a-f0-9]{8})\\Z",
         "type": "string"
       },
       "InstanceId": {
         "description": "The ID of the instance that backs the WordPress site.",
         "type": "string"
       },
       "PublicIp": {
         "description": "The public IP for the WordPress site.",
         "type": "string"
       }
     },
     "required": [ "Name", "SubnetId" ],
     "primaryIdentifier": [ "/properties/PublicIp", "/properties/InstanceId" ],
     "readOnlyProperties": [ "/properties/PublicIp", "/properties/InstanceId" ],
     "additionalProperties": false
   }
   ```

1. Validate the schema\.

   ```
   $ cfn validate
   ```

1. Update the auto\-generated files in the resource provider package to view the resource provider schema updates\. Upon initiation of the resource provider project, the AWS CLI generates supporting files and code for the resource provider\. Regenerate the code to see the updated schema\.

   ```
   $ cfn generate
   ```
**Note**  
When using Maven, as part of the build process the `generate` command is automatically run before the code is compiled\. So your changes will never get out of sync with the generated code\.  
Be aware the CloudFormation CLI must be in a location Maven/the system can find\. For more information, see [Setting up your environment for developing extensions](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/what-is-cloudformation-cli.html#resource-type-setup)\.

For more information on the whole process, see [Modeling Resource Providers for Use in AWS CloudFormation](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/resource-type-model.html)\.

## Step 3: Generate Artifacts<a name="generateartifactsforcfnsubmit"></a>

Run the following command to generate artifacts for `cfn submit`\.

```
$ mvn package
```

## Step 4: Register Your Resource<a name="customresources-registerresource"></a>

AWS Config does not require resource provider handlers to perform configuration tracking for your resource\. Run the following command to register your resource\.

```
$ cfn submit
```

For more information, see [Registering Resource Providers for Use in AWS CloudFormation Templates](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/resource-type-register.html)\.

## Step 5: Publish Resource Configuration<a name="customresources-publishresourceconfiguration"></a>

Determine the configuration for MyCustomNamespace::Testing::WordPress\.

```
{
  "Name": "MyWordPressSite",
  "SubnetId": "subnet-abcd0123",
  "InstanceId": "i-01234567",
  "PublicIp": "my-wordpress-site.com"
}
```

Determine the schema version id from AWS CloudFormation `DescribeType`\. 

In the AWS Config see if this resource configuration is accepted\. To evaluate compliance you can write AWS Config rules using this resource\. For additional information, see [Record and Delete a Configuration State for Third\-Party Resources Using AWS CLI](https://docs.aws.amazon.com/config/latest/developerguide/customresources-cli.html)\. 

Optional: To automate recording of configuration, implement a periodic or change\-based configuration collectors\.

## Region Support<a name="custom-resources-regions"></a>

This feature is supported in the following regions:


****  
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/config/latest/developerguide/customresources.html)
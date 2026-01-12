## Architecture overview

This project creates a .NET 8 AWS Serverless project that deploys one or more AWS Lambda functions. The AWS Lambda functions are defined in the
`serverless.template` AWS CloudFormation template file. Additional AWS resources can be defined in the `serverless.template` file, which is created as
part of the deployment workflow.

The project uses the .NET Global Tool <a href="https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools/" target="_blank">
Amazon.Lambda.Tools</a> to deploy. The deploy process builds the .NET project and creates a zip file of the publish .NET binaries. The zip file is
uploaded to Amazon S3 and the `serverless.template` is updated to point to the location in Amazon S3. The `serverless.template` file is then sent to
AWS CloudFormation to create the defined AWS resources in the AWS CloudFormation template file.

## Connections and permissions

The Amazon CodeCatalyst environment requires an AWS account connection for your Amazon CodeCatalyst space and a configured IAM role for your project
workflow. You can create a new account connection from the AWS accounts menu in your Amazon CodeCatalyst space. AWS IAM roles added to the account
extension are used to authorize project workflows to access AWS account resources. This blueprint requires the following IAM role permissions and
trust policy:

### Trust policy

```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "CodeCatalyst",
          "Effect": "Allow",
          "Principal": {
              "Service": [
                  "codecatalyst-runner.amazonaws.com",
                  "codecatalyst.amazonaws.com"
              ]
          },
          "Action": "sts:AssumeRole"
      }
  ]
}
```

### IAM role

The required permissions for the Deployment role depend on the project type you selected when creating the project and any additional resources added
to the `serverless.template`. Below are the minimum permissions needed to deploy the defined AWS Lambda functions in the `serverless.template`:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:GetObject",
                "iam:PassRole",
                "iam:DeleteRole",
                "iam:GetRole",
                "iam:TagRole",
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "cloudformation:*",
                "lambda:*",
                "apigateway:*"  // Needed if the event type for the Lambda function is "Api" or "HttpApi".
            ],
            "Resource": "*"
        }
    ]
}
```

## Build and test project

To build the .NET project from the command line execute the following command in project top directory.

```
dotnet build ServerlessApp.sln -c Release
```

The following command can be used to execute the test for the project.

```
dotnet test ServerlessApp.sln -c Release —no-build —logger "trx"
```

## Deploying to AWS

If a deployment role was specified during project creation then the `main.yaml` workflow will be preconfigured to deploy the serverless application to AWS on Git pushes to the main branch. The project can also be deployed from local development environments to AWS.

**Note** - You need to install the  `zip`  CLI tool on Linux OS as it is required to maintain Linux file permissions. 

To install zip on Linux OS, run the following commands depending on your distribution's package management tool.

For distributions using  `apt-get`:

```
sudo apt-get install zip
```

For distributions using  `yum`:

```
sudo yum install zip
```


### From Visual Studio

Using the <a href="(https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.AWSToolkitforVisualStudio2022" target="_blank">AWS Toolkit for Visual Studio</a> the project can be deployed to AWS. The toolkit uses the same <a href="https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools/" target="_blank"> Amazon.Lambda.Tools </a> tooling as the project's workflow.

After installing the toolkit the project can be deployed to AWS by right clicking on the project in the solution explorer and selecting **Publish to AWS Lambda**.


After installing the toolkit, the project can be deployed to AWS by right clicking on the project in the solution explorer and selecting **Publish to AWS Lambda**. The toolkit will then ask you to input a CloudFormation stack name and an S3 bucket name where the build artifacts will be stored. When you are ready to deploy click the **Publish** button.


### From .NET CLI

Install <a href="https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools/" target="_blank">Amazon.Lambda.Tools </a> if not already installed.

```
dotnet tool install -g Amazon.Lambda.Tools
```

If already installed check if new version is available.

```
dotnet tool update -g Amazon.Lambda.Tools
```

In the serverless application's project directory run the following command to initiate a deployment.

```
dotnet lambda deploy-serverless --region <REGION> --stack-name <STACK_NAME> --s3-bucket <BUCKET_NAME>
```

You can replace the `--s3-bucket <BUCKET_NAME>` with `--resolve-s3 true` to automatically create an S3 bucket that stores the build artifacts.


## Project resources

This blueprint creates the following Amazon CodeCatalyst resources:

- `.codecatalyst/workflows/main.yaml` - Workflow that pushes to the main branch. The workflow builds the project and runs the project's tests. If a
  deployment role was specified during the project creation, the workflow will also use the
  <a href="https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools/" target="_blank">Amazon.Lambda.Tools </a> to deploy your
  serverless application to AWS.
- `.codecatalyst/workflows/pull-request.yaml` - Workflow for pull requests on any branch. The workflow builds the project and runs the project's
  tests.
- `src` - Folder that contains the .NET 8 serverless application source code. It also contains the `serverless.template` file used by AWS
  CloudFormation to provision resources for your application.
- `tests` - Folder that contains unit tests for the project. The unit test project uses the <a href="https://xunit.net/" target="_blank">xUnit.net</a>
  test framework. For more information on source repositories, see the _Working with source repositories_ section in the **Amazon CodeCatalyst User Guide**.
- Workflows defined in `.codecatalyst/workflows`

  A workflow is an automated procedure that defines how to build, test, and deploy the serverless application. For more information, see the _Build, test, and deploy with workflows_ section of the **Amazon CodeCatalyst User Guide**

- Environment(s) - An abstraction of infrastructure resources for deploying applications. Environments can be used to organize deployment actions into
  a production or non-production environment.

  For more information on environments, see the _Organizing deployments using environments_ section in the **Amazon CodeCatalyst User Guide**

- Dev Environment - A cloud-based development environment. You must manually create a Dev Environment with the generated devfile using the Create Dev
  Environment operation on Amazon CodeCatalyst.

  For more information on creating Dev Environments, see the _Working with Dev Environments_ section in the **Amazon CodeCatalyst User Guide**

## Additional resources

See the Amazon CodeCatalyst User Guide for additional information on using the features and resources of Amazon CodeCatalyst.

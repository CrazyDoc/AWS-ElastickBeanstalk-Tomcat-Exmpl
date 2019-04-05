# Deploy a CapBPM environment on AWS using automation

## CapBPM on AWS architecture and features
The features of using this architecture are as follows:
* A complete CapBPM environment is automatically deployed in about 20 minutes.
* CapBPM is deployed in an isolated Virtual Private Cloud
* The environment enables automatic scaling up and down based on load
* Managed services are used that provide automated patching and maintenance of OS, middleware, and database software
* Database backups are performed automatically to enable operational and disaster recovery

A high-level diagram showing how the different functions of CapBPM map to AWS Services is shown below.
![Scheme](Scheme.png?raw=true "Scheme")

**AWS Elastic Beanstalk** is used to deploy the application onto Linux Tomcat with Java servers.  Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications. It covers everything from capacity provisioning, load balancing, regular OS and middleware updates, autoscaling, and high availability, to application health monitoring.

**Amazon Relational Database Service (RDS)** is used to provide database for CapBPM.

**Amazon Simple Storage Service (S3)** is used as a file repository.  S3 is designed to deliver 99.999999999% durability, and stores data for millions of applications used by market leaders in every industry. S3 provides comprehensive security and compliance capabilities that meet even the most stringent regulatory requirements.

**Amazon Simple Notification Service (SNS)** is used enable to send emails to users. SNS is a highly available, durable, secure, fully managed pub/sub messaging service.

### Deployment Instructions
0. This template must be run by an AWS IAM User who has sufficient permission to create the required resources.  These resources include:  VPC, IAM User and Roles, S3 Bucket, EC2 Instance, ALB, Elastic Beanstalk.  If you are not an Administrator of the AWS account you are using, please check with them before running this template to ensure you have sufficient permission.  

1. From your AWS account, [open the CloudFormation Management Console](https://console.aws.amazon.com/cloudformation/) and choose **Create Stack**.  From there upload "infrastructure-beanstalk/cd/CFstack.json", and choose **Next**.

2. On the next screen, provide a **Stack Name** and a few other parameters for your environment. When you've provided appropriate values for the **Parameters**, choose **Next**.

3. On the next screen, you can provide some other optional information like tags at your discretion, or just choose **Next**.

4. On the next screen, you can review what will be deployed. At the bottom of the screen, there is a check box for you to acknowledge that **AWS CloudFormation might create IAM resources with custom names**. This is correct; the template being deployed creates four custom roles that give permission for the AWS services involved to communicate with each other. Details of these permissions are inside the CloudFormation template referenced in the URL given in the first step. Check the box acknowledging this and choose **Next**.

5. You can watch as CloudFormation builds out your CapBPM environment. A CloudFormation deployment is called a *stack*. The parent stack creates several child stacks depending on the parameters you provided.  When all the stacks have reached the green CREATE_COMPLETE status, as shown in the screenshot following, then the CapBPM architecture has been deployed.  Select the **Outputs** tab to find your environment URL.

6. After clicking on the provided URL, you will be taken to the Camunda login screen. Make sure that [PipeLine](https://console.aws.amazon.com/codesuite/codepipeline/pipelines?) managed to this point to complete the first deploy.

### Ongoing Operations
At this point, you have a fully functioning CapBPM environment to begin using.  Following are some helpful points to consider regarding how to support this environment on-going.

#### SSL
Add an [SSL certificate](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-elb.html) to your ALB. This can be created by your certification authority or by using [AWS ACM](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html).

#### Web Security
Consider implementing a [AWS Web Application Firewall (WAF)](https://aws.amazon.com/waf/) in front of your application to help protect against common web exploits that could affect availability, compromise security or consume excessive resources.  You can use AWS WAF to create custom rules that block common attack patterns, such as SQL injection or cross-site scripting.  Learn more in the whitepaper ["Use AWS WAF to Mitigate OWASP’s Top 10 Web Application Vulnerabilities"](https://d0.awsstatic.com/whitepapers/Security/aws-waf-owasp.pdf) .  You can deploy AWS WAF on either Amazon CloudFront as part of your CDN solution or on the Application Load Balancer (ALB) that was deployed as a part of this solution.

#### Pull logs, view monitoring data, get alerts, and apply patches
Using the Elastic Beanstalk service you can [pull log files from one or more of your instances](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.logging.html).  You can also [view monitoring data](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-health.html) including CPU utilization, network utilization, HTTP response codes, and more.  From this monitoring data, you can [configure alarms](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.alarms.html) to notify you of events within your application environment.  Elastic Beanstalk also makes [managed platform updates available](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-platform-update-managed.html), including Linux, Tomcat, Java, and Nginx upgrades that you can apply during maintanence windows your define.

Using AWS Relational Database Services (RDS) you can [pull log files from you database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.html).  You can also [view monitoring data and create alarms on metrics](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Aurora.Monitoring.html) including disk space, CPU utilization, memory utilization, and more. RDS also makes available [DB Engine upgrades](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/) that are applied automatically during a maintenance window you define.

#### Access Running Instances
[Crate SSH key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair) or [Import SSH key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws). Add your key in EB enviroment in [Environment Configuration the Elastic Beanstalk Console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-console.html).
Now if you need to access the command line on the running instances, this can be done by using the [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html). It lets you manage your Amazon EC2 instances through an interactive one-click browser-based shell or through the AWS CLI. Session Manager provides secure and auditable instance management without the need to open inbound ports, maintain bastion hosts, or manage SSH keys.  Just go to the the [AWS Systems Manager Session Manager Console](https://console.aws.amazon.com/systems-manager/session-manager/) and **click Start session**.  You'll then see a list of your instances. Select the one that you want to access and **click Start session**.  Now you have a shell with sudoers access.

#### Fault tolerance and backups
[Save the application configuration](https://docs.amazonaws.cn/en_us/elasticbeanstalk/latest/dg/environment-configuration-savedconfig.html) - This will help you to quickly restore the app. Elastic Beanstalk keeps a highly available copy of your current and previous application versions as well as your environment configuration.  This can be used to re-deploy or re-create your application environment at any time and serves as a 'backup'.  You can also [clone an Elastic Beanstalk environment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.clone.html) if you want a temporary environment for testing or as a part of a recovery exercise.  High availability and fault tolerance are provided are achieved by [configuring your environment to have a minimum of 2 instances](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.as.html).

The AWS Relational Database Service (RDS) automatically takes backups of your database that you can use to restore or "roll-back" the state of your application.  You can [configure how long they are retained and when they are taken using the RDS management console](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html).  These backups can also be used to create a copy of your database for use in testing or as a part of a recovery exercise.

#### Scalability
Your Elastic Beanstalk environment is configured to scale automatically based on CPU utilization within the minimum and maximum instance parameters(default 20 - 60 CPU percent utilization). [You can change this configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.as.html) to specify a larger minimum footprint, a larger maximum footprint, different instance sizes, or different scaling parameters.  This allows your application environment to respond automatically to the amount of load seen from your users.

Your Relational Database Environment can be scaled by [selecting a larger instance type](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/) or increasing the storage capacity of your instances.

### Changing your Camunda & Tomcat configuration

Elastic Beanstalk обеспечивает эластичную среду для приложений, которые могут динамически увеличиваться и уменьшаться в зависимости от нагрузки. Вы также можете использовать Elastic Beanstalk для репликации или повторного создания среды приложения. Когда Elastic Beanstalk делает это, он использует файлы конфигурации, чтобы обеспечить синхронизацию всех экземпляров, которые он динамически создает. Эта конфигурация хранится в специальном каталоге в пакете приложения, который вы предоставляете Elastic Beanstalk [называемый '.ebextensions'] (https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html). Just change the settings using .ebextensions. The structure of the native file.

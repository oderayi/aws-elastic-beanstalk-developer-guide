# Configuring an Application Load Balancer<a name="environments-cfg-alb"></a>

When you [enable load balancing](using-features-managing-env-types.md#using-features.managing.changetype), your AWS Elastic Beanstalk environment is equipped with an Elastic Load Balancing load balancer to distribute traffic among the instances in your environment\. Elastic Load Balancing supports a few load balancer types\. To learn about them, see the [Elastic Load Balancing User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)\. 

This topic describes the configuration of an [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)\. For information about configuring all the load balancer types that Elastic Beanstalk supports, see [Load Balancer for Your AWS Elastic Beanstalk Environment](using-features.managing.elb.md)\.

**Note**  
You can choose the type of load balancer that your environment uses only during environment creation\. You can change settings to manage the behavior of your running environment's load balancer, but you can't change its type\.

## Introduction<a name="environments-cfg-alb-intro"></a>

An Application Load Balancer inspects traffic at the application network protocol layer to identify the request's path so that it can direct requests for different paths to different destinations\.

By default, an Application Load Balancer performs the same function as a Classic Load Balancer\. The default listener accepts HTTP requests on port 80 and distributes them to the instances in your environment\. You can add a secure listener on port 443 with a certificate to decrypt HTTPS traffic, configure health check behavior, and push access logs from the load balancer to an Amazon Simple Storage Service \(Amazon S3\) bucket\.

**Note**  
Unlike a Classic Load Balancer or a Network Load Balancer, an Application Load Balancer can't have transport layer \(layer 4\) TCP or SSL/TLS listeners\. It supports only HTTP and HTTPS listeners\. Additionally, it can't use backend authentication to authenticate HTTPS connections between the load balancer and backend instances\.

In an Elastic Beanstalk environment, you can use an Application Load Balancer to direct traffic for certain paths to a different port on your web server instances\. With a Classic Load Balancer, all traffic to a listener is routed to a single port on the backend instances\. With an Application Load Balancer, you can configure multiple *rules* on the listener to route requests to certain paths to different backend ports\.

For example, you could run a login process separately from your main application\. While the main application on your environment's instances accepts the majority of requests and listens on port 80, your login process listens on port 5000 and accepts requests to the `/login` path\. All incoming requests from clients come in on port 80\. With an Application Load Balancer, you can configure a single listener for incoming traffic on port 80, with two rules that route traffic to two separate processes, depending on the path in the request\. One rule routes traffic to `/login` to the login process listening on port 5000\. The default rule routes all other traffic to the main application process listening on port 80\.

An Application Load Balancer rule maps a request to a *target group*\. In Elastic Beanstalk, a target group is represented by a *process*\. You can configure a process with a protocol, port, and health check settings\. The process represents the process running on the instances in your environment\. The default process is a listener on port 80 of the reverse proxy \(nginx or Apache\) that runs in front of your application\.

**Note**  
Outside of Elastic Beanstalk, a target group maps to a group of instances\. A listener can use rules and target groups to route traffic to different instances based on the path\. Within Elastic Beanstalk, all of your instances in your environment are identical, so the distinction is made between processes listening on different ports\.

A Classic Load Balancer uses a single health check path for the entire environment\. With an Application Load Balancer, each process has a separate health check path that is monitored by the load balancer and Elastic Beanstalk\-enhanced health monitoring\.

To use an Application Load Balancer, your environment must be in a default or custom VPC, and must have a service role with the standard set of permissions\. If you have an older service role, you might need to [update the permissions](iam-instanceprofile.md#iam-instanceprofile-addperms) on it to include `elasticloadbalancing:DescribeTargetHealth` and `elasticloadbalancing:DescribeLoadBalancers`\. For more information about Application Load Balancers, see the [User Guide for Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)\.

**Note**  
The Application Load Balancer health check doesn't take into account the Elastic Beanstalk health check path\. Instead, it uses the specific path configured for each process separately\.

## Configuring an Application Load Balancer Using the Elastic Beanstalk Console<a name="environments-cfg-alb-console"></a>

You can use the Elastic Beanstalk console to configure an Application Load Balancer's listeners, processes, and rules, during environment creation or later when your environment is running\.

**To configure an Application Load Balancer in the Elastic Beanstalk console during environment creation**

1. Use the [Create New Environment wizard](environments-create-wizard.md) to start creating your environment\.

1. On the wizard's main page, before choosing **Create environment**, choose **Configure more options**\.

1. Choose the **High availability** configuration preset\.

   Alternatively, on the **Capacity** configuration card, configure a **Load balanced** environment type\. For details, see [Capacity](environments-create-wizard.md#environments-create-wizard-capacity)\.

1. On the **Load balancer** configuration card, choose **Modify**\.

1. Select the **Application Load Balancer** option, if it isn't already selected\.  
![\[Elastic Load Balancing configuration page - choosing load balancer type\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-type-chooser.png)

1. Make any Application Load Balancer configuration changes that your environment requires\.

1. Choose **Save**, and then make any other configuration changes that your environment requires\.

1. Choose **Create environment**\.

**To configure a running environment's Application Load Balancer in the Elastic Beanstalk console**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management page](environments-console.md) for your environment\.

1. Choose **Configuration**\.

1. On the **Load balancer** configuration card, choose **Modify**\.
**Note**  
If the **Load balancer** configuration card doesn't have a **Modify** button, your environment doesn't have a load balancer\. To learn how to set one up, see [Changing Environment Type](using-features-managing-env-types.md#using-features.managing.changetype)\.

1. Make the Application Load Balancer configuration changes that your environment requires\.

1. Choose **Apply**\.

**Topics**
+ [Listeners](#environments-cfg-alb-console-listeners)
+ [Processes](#environments-cfg-alb-console-processes)
+ [Rules](#environments-cfg-alb-console-rules)
+ [Access Log Capture](#environments-cfg-alb-console-logs)
+ [Example: Application Load Balancer with a Secure Listener and Two Processes](#environments-cfg-alb-console-example)

### Listeners<a name="environments-cfg-alb-console-listeners"></a>

Use this list to specify listeners for your load balancer\. Each listener routes incoming client traffic on a specified port using a specified protocol to one or more processes on your instances\. Initially, the list shows the default listener, which routes incoming HTTP traffic on port 80 to a process named **default**, which listens to HTTP port 80\.

![\[Application Load Balancer configuration - listener list\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-listeners.png)

**To configure an existing listener**

1. Select the check box next to its table entry, and then choose **Actions**, **Edit**\.

1. If you chose **Edit**, use the **Application Load Balancer listener** dialog box to edit settings, and then choose **Save**\.

**To add a listener**

1. Choose **Add listener**\.

1. In the **Application Load Balancer listener** dialog box, configure settings you want, and then choose **Add**\.

Use the **Application Load Balancer listener** dialog box settings to choose the port and protocol on which the listener listens to traffic\. If you choose the HTTPS protocol, configure SSL settings\.

Before you configure an HTTPS listener, ensure that you have a valid SSL certificate\. Create a new certificate using AWS Certificate Manager \(ACM\), or upload a certificate and key to AWS Identity and Access Management \(IAM\)\. For more information about requesting an ACM certificate, see [Request a Certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request.html) in the *AWS Certificate Manager User Guide*\. For more information about importing third\-party certificates into ACM, see [Importing Certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\. If ACM isn't [available in your AWS Region](http://docs.aws.amazon.com/general/latest/gr/rande.html#acm_region), upload your existing certificate and key to IAM\.

For more information about creating and uploading certificates to IAM, see [Working with Server Certificates](https://docs.aws.amazon.com/IAM/latest/UserGuide/ManagingServerCerts.html) in *IAM User Guide*\. For more detail on configuring HTTPS and working with certificates in Elastic Beanstalk, see [Configuring HTTPS for your Elastic Beanstalk Environment](configuring-https.md)\.

### Processes<a name="environments-cfg-alb-console-processes"></a>

Use this list to specify processes for your load balancer\. A process is a target for listeners to route traffic to\. Each listener routes incoming client traffic on a specified port using a specified protocol to one or more processes on your instances\. Initially, the list shows the default process, which listens to incoming HTTP traffic on port 80\.

![\[Application Load Balancer configuration - process list\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-processes.png)

You can edit the settings of an existing process, or add a new process\. To start editing a process on the list or adding a process to it, use the same steps listed for the [listener list](#environments-cfg-alb-console-listeners)\. The **Environment process** dialog box opens\.

**Topics**
+ [Definition](#environments-cfg-alb-console-process-definition)
+ [Health Check](#environments-cfg-alb-console-process-healthchecks)
+ [Sessions](#environments-cfg-alb-console-process-sessions)

#### Definition<a name="environments-cfg-alb-console-process-definition"></a>

Use these settings to define the process: its **Name**, and the **Port** and **Protocol** on which it listens to requests\.

![\[Application Load Balancer process settings for name, port, and protocol\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-process-definition.png)

#### Health Check<a name="environments-cfg-alb-console-process-healthchecks"></a>

Use the following settings to configure process health checks:
+ **HTTP code** – The HTTP status code designating a healthy process\.
+ **Path** – The health check request path for the process\.
+ **Timeout** – The amount of time, in seconds, to wait for a health check response\.
+ **Interval** – The amount of time, in seconds, between health checks of an individual instance\. The interval must be greater than the timeout\.
+ **Unhealthy threshold**, **Healthy threshold** – The number of health checks that must fail or pass, respectively, before Elastic Load Balancing changes an instance's health state\.
+ **Deregistration delay** – The amount of time, in seconds, to wait for active requests to complete before deregistering an instance\.

![\[Application Load Balancer process settings for health check\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-process-healthcheck.png)

**Note**  
The Elastic Load Balancing health check doesn't affect the health check behavior of an environment's Auto Scaling group\. Instances that fail an Elastic Load Balancing health check will not automatically be replaced by Amazon EC2 Auto Scaling unless you manually configure Amazon EC2 Auto Scaling to do so\. See [Auto Scaling Health Check Setting](environmentconfig-autoscaling-healthchecktype.md) for details\. 

For more information about health checks and how they influence your environment's overall health, see [Basic Health Reporting](using-features.healthstatus.md)\.

#### Sessions<a name="environments-cfg-alb-console-process-sessions"></a>

Use these settings to enable or disable sticky sessions \(**Stickiness policy enabled**\) and to configure a sticky session's duration \(**Cookie duration**\), up to **604800** seconds\.

![\[Application Load Balancer process settings for session sticking\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-process-sessions.png)

### Rules<a name="environments-cfg-alb-console-rules"></a>

Use this list to specify listener rules for your load balancer\. A rule maps requests that the listener receives on a specific path pattern to a target process\. Each listener can have multiple rules, routing requests on different paths to different processes on your instances\. Rules have numeric priorities that determine the precedence in which they are applied to incoming requests\. For each new listener you add, Elastic Beanstalk adds a default rule that routes all the listener's traffic to the default process\. The default rule's precedence is the lowest; it's applied if no other rule for the same listener matches the incoming request\. Initially, the list shows the default rule of the default HTTP port 80 listener\.

![\[Application Load Balancer configuration - rule list\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-rules.png)

You can edit the settings of an existing rule, or add a new rule\. To start editing a rule on the list or adding a rule to it, use the same steps listed for the [listener list](#environments-cfg-alb-console-listeners)\. The **Listener rule** dialog box opens, with the following settings:
+ **Name** – The rule's name\.
+ **Listener port** – The port of the listener that the rule applies to\.
+ **Priority** – The rule's priority\. A lower priority number has higher precedence\. Priorities of a listener's rules must be unique\.
+ **Path pattern** – A pattern defining the request paths that the rule applies to\.
+ **Process** – The process to which the load balancer routes requests that match the rule\.

When editing any existing rule, you can't change its **Name** and **Listener port**\. When editing a default rule, **Process** is the only setting you can change\.

![\[Application Load Balancer configuration - rule list\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-rule-dialog.png)

### Access Log Capture<a name="environments-cfg-alb-console-logs"></a>

Use these settings to configure Elastic Load Balancing to capture logs with detailed information about requests sent to your Application Load Balancer\. Access log capture is disabled by default\. When enabled \(**Store logs**\), Elastic Load Balancing stores the logs in the Amazon S3 bucket that you configure \(**S3 bucket**\)\. The **Prefix** setting specifies a top\-level folder in the bucket for the logs\. Elastic Load Balancing places the logs in a folder named `AWSLogs` under your prefix\. If you don't specify a prefix, Elastic Load Balancing places its folder at the root level of the bucket\.

![\[Application Load Balancer configuration - access logs\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-logs.png)

### Example: Application Load Balancer with a Secure Listener and Two Processes<a name="environments-cfg-alb-console-example"></a>

In this example, your application requires end\-to\-end traffic encryption and a separate process for handling administrative requests\. To configure your environment's Application Load Balancer to meet these requirements, you remove the default listener, add an HTTPS listener, indicate that the default process listens to port 443 on HTTPS, and add a process and a rule for admin traffic on a different path\.

**To configure the load balancer for this example**

1. *Remove the default port 80 HTTP listener\.* Select the default listener, and then, for **Actions**, choose **Mark as 'removed'**\.  
![\[Application Load Balancer configuration - removing default listener\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-listeners-removed.png)

1. *Add a secure listener\.* For **Port**, type `443`\. For **Protocol**, choose `HTTPS`\. For **SSL certificate**, choose the ARN of your SSL certificate\. For example, `arn:aws:iam::123456789012:server-certificate/abc/certs/build`, or `arn:aws:acm:us-east-2:123456789012:certificate/12345678-12ab-34cd-56ef-12345678`\.  
![\[Application Load Balancer configuration - adding a secure listener\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-listeners-https.png)

   You can now see your additional listener on the list\.  
![\[Application Load Balancer configuration - listener list with two listeners\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-listeners2.png)

1. *Configure the default process to HTTPS\.* Select the default process, and then, for **Actions**, choose **Edit**\. For **Port**, type `443`\. For **Protocol**, choose `HTTPS`\.  
![\[Application Load Balancer configuration - configuring default process to HTTPS\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-process-definition-https.png)

1. *Add an admin process\.* For **Name**, type `admin`\. For **Port**, type `443`\. For **Protocol**, choose `HTTPS`\. Under **Health check**, for **Path**, type `/admin`\.  
![\[Application Load Balancer configuration - adding admin process\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-process-definition-https-admin.png)

1. *Add a rule for admin traffic\.* For **Name**, type `admin`\. For **Listener port**, type `443`\. For **Path pattern**, type `/admin/*`\. For **Process**, choose `admin`\.  
![\[Application Load Balancer configuration - adding admin rule\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-alb-rule-https-admin.png)

## Configuring an Application Load Balancer Using the EB CLI<a name="environments-cfg-alb-ebcli"></a>

The EB CLI prompts you to choose a load balancer type when you run [`eb create`](eb3-create.md)\.

```
$ eb create
Enter Environment Name
(default is my-app): test-env
Enter DNS CNAME prefix
(default is my-app): test-env-DLW24ED23SF

Select a load balancer type
1) classic
2) application
3) network
(default is 1): 2
```

You can also specify a load balancer type with the `--elb-type` option\.

```
$ eb create test-env --elb-type application
```

## Application Load Balancer Namespaces<a name="environments-cfg-alb-namespaces"></a>

You can find settings related to Application Load Balancers in the following namespaces:
+ `[aws:elasticbeanstalk:environment](command-options-general.md#command-options-general-elasticbeanstalkenvironment)` – Choose the load balancer type for the environment\. The value for an Application Load Balancer is `application`\.
+ `[aws:elbv2:loadbalancer](command-options-general.md#command-options-general-elbv2)` – Configure access logs and other settings that apply to the Application Load Balancer as a whole\.
+ `[aws:elbv2:listener](command-options-general.md#command-options-general-elbv2-listener)` – Configure listeners on the Application Load Balancer\. These settings map to the settings in `aws:elb:listener` for Classic Load Balancers\.
+ `[aws:elbv2:listenerrule](command-options-general.md#command-options-general-elbv2-listenerrule)` – Configure rules that route traffic to different processes, depending on the request path\. Rules are unique to Application Load Balancers\.
+ `[aws:elasticbeanstalk:environment:process](command-options-general.md#command-options-general-environmentprocess)` – Configure health checks and specify the port and protocol for the processes that run on your environment's instances\. The port and protocol settings map to the instance port and instance protocol settings in `aws:elb:listener` for a listener on a Classic Load Balancer\. Health check settings map to the settings in the `aws:elb:healthcheck` and `aws:elasticbeanstalk:application` namespaces\.

**Example \.ebextensions/application\-load\-balancer\.config**  
To get started with an Application Load Balancer, use a [configuration file](ebextensions.md) to set the load balancer type to `application`\.  

```
option_settings:
  aws:elasticbeanstalk:environment:
    LoadBalancerType: application
```

**Note**  
You can set the load balancer type only during environment creation\.

**Example \.ebextensions/alb\-access\-logs\.config**  
The following configuration file enables access log uploads for an environment with an Application Load Balancer\.  

```
option_settings:
  aws:elbv2:loadbalancer:
    AccessLogsS3Bucket: my-bucket
    AccessLogsS3Enabled: 'true'
    AccessLogsS3Prefix: beanstalk-alb
```

**Example \.ebextensions/alb\-default\-process\.config**  
The following configuration file modifies health check and stickiness settings on the default process\.  

```
option_settings:
  aws:elasticbeanstalk:environment:process:default:
    DeregistrationDelay: '20'
    HealthCheckInterval: '15'
    HealthCheckPath: /
    HealthCheckTimeout: '5'
    HealthyThresholdCount: '3'
    UnhealthyThresholdCount: '5'
    Port: '80'
    Protocol: HTTP
    StickinessEnabled: 'true'
    StickinessLBCookieDuration: '43200'
```

**Example \.ebextensions/alb\-secure\-listener\.config**  
The following configuration file adds a secure listener and a matching process on port 443\.  

```
option_settings:
  aws:elbv2:listener:443:
    DefaultProcess: https
    ListenerEnabled: 'true'
    Protocol: HTTPS
    SSLCertificateArns: arn:aws:acm:us-east-2:123456789012:certificate/21324896-0fa4-412b-bf6f-f362d6eb6dd7
  aws:elasticbeanstalk:environment:process:https:
    Port: '443'
    Protocol: HTTPS
```

**Example \.ebextensions/alb\-admin\-rule\.config**  
The following configuration file adds a secure listener with a rule that routes traffic with a request path of `/admin` to a process named `admin` that listens on port 4443\.  

```
option_settings:
  aws:elbv2:listener:443:
    DefaultProcess: https
    ListenerEnabled: 'true'
    Protocol: HTTPS
    Rules: admin
    SSLCertificateArns: arn:aws:acm:us-east-2:123456789012:certificate/21324896-0fa4-412b-bf6f-f362d6eb6dd7
  aws:elasticbeanstalk:environment:process:https:
    Port: '443'
    Protocol: HTTPS
  aws:elasticbeanstalk:environment:process:admin:
    HealthCheckPath: /admin
    Port: '4443'
    Protocol: HTTPS
  aws:elbv2:listenerrule:admin:
    PathPatterns: /admin/*
    Priority: 1
    Process: admin
```
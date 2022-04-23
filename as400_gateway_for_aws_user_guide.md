# Introduction

Infoview Systems AS400Gateway suite of products eliminates the stress and impact of IBM i / AS400 legacy system integration on development teams, minimizes the time and resources put into building integrations by hand, and enables non-IBM i developers to unlock legacy business logic and data directly from the comfort of their modern integration development stack. AS400Gateway for AWS is an Amazon Machine Image (AMI) that provides generic REST API integration for IBM i operations (Program call, Service Program call, Command call, Data Queue Read and Write). In addition the product can be configured to continuously listen for Data Queue messages and forward them to AWS SQS or SNS for further processing.

The product provides a convenient out of the box IBM i integration capabilities for companies running their enterprise integrations on AWS stack. Certified and rigorously tested by AWS marketplace team, the connector was designed to accelerate IBM i / AS400 integrations with other systems and services. In addition to the core product available to customers in AMI format, we also provide a reference AWS architecture that includes a sample API and event based use cases.

We are a global cross-platform service team with a unique fluency in legacy and modern technology stacks, including Mulesoft and IBM i / AS400. Infoview&#39;s dedicated customer success representative coordinates just-in-time technical assistance and support to client teams ensuring you have all the help you may need, when you need it. We are more than happy to provide a trial license for our products, participate in discovery sessions, run live demos for typical integration scenarios with our Gateway products, as well as assist with or perform a proof of concept based on particular use cases.

[Contact us](http://www.infoviewsystems.com/contact-us) for connector pricing info, trial license, or support questions.

# AS400Gateway for AWS Overview

The IBM AS/400 was first introduced in 1988 and evolved into a very stable modern all-purpose integrated solution that requires little or no management. The system is able to run core line of business applications securely and predictably, focusing on quality of service and availability and offering a compelling total cost of ownership for integrated on-premises solutions. IBM made several changes to the server and OS name (iSeries, System i, IBM i, Power Systems for i) but most still refer to it as AS/400.

The IBM i platform offers a number of integration options including PHP, Java, WebSphere, specialized lightweight web service containers, FTP, SMTP / emails, DB2 interfaces, data queues, integrated file systems - IFS, as well as number of products offered by IBM and Third party vendors. The main benefit of using &quot;native&quot; options such as Program Calls and Data Queues is that integration or cloud development team does not have to acquire in-depth knowledge of IBM i technologies in order to build integration layer, and can easily communicate with external systems using only traditional development tools.

Program Call is the most straightforward and low code option for exposing IBM i business logic as a reusable API. The AS400Gateway for AWS provides a generic API interface to call any program on IBM i, passing parameters into the program and receiving the results back in real time.

Data queues are native IBM i objects designed primarily for inter-process communications. They are lightweight persistent queues that support processing by a key, FIFO or LIFO. The majority of integration use cases can be implemented with the pair of request and response Data Queues. Source system places a message to request data queue and waits for acknowledgement message on response data queue. The target system receives and processes a message from the request data queue then places the acknowledgement to the response data queue.

The AS400Gateway for AWS provides the REST API to read or write the messages into Data Queue on demand, or continuously listen for new Data Queue messages and forward to AWS SQS / SNS in near real time.

# IBM i Prerequisites

- IBM i OS version:V5R4 and higher
- AWS VPC where the AMI is running must be able to reach the IBM i servers on ports 446, 449, 8470, 8472,8473,8475 and 8476 for non-SSL communications, and ports 448, 449, 9470, 9472, 9473, 9475 and 9476 accessible for SSL communications.
- IBM i must have **\*CENTRAL, \*DTAQ, \*RMTCMD, \*SIGNON and \*SRVMAP** host servers running in the QSYSWRK subsystem
- If secure TLS connection is used, the TLS certificate must be applied to Central, Data Queue, Remote Command, File, Signon, and DDM / DRDA services in Digital Certificate Manager
- IBM i user ID must be authorized to perform the operations on the intended IBM i objects
- If there&#39;s an additional security software that locks down the remote execution functionality, the IBM i user ID defined for connector configuration must be allowed to execute remote calls and access database, IFS and DDM services

# AWS Reference Architecture

Below is a sample AWS architecture that includes various services and components typically used when implementing cloud integration solution for bidirectional integrations with IBM i based back-end systems.

Salesforce is used as a sample external system that sends orders to IBM i based ERP and receives order statuses back from the ERP in near real time. Note that most companies will likely have different services and applications used for API management, security policies, token validations, routing and other AWS components interacting with AS400Gateway for AWS.

![](RackMultipart20220423-1-c0u157_html_babcd4b037181062.png)

# AWS setup guide

Here are the following AWS services for implementing the solution

| **#** | **Resource Name** | **Qty** |
| --- | --- | --- |
| 1 | VPC | 1 |
| 2 | Subnets | 2 |
| 3 | EC2 Instance | 2 |
| 4 | NLB | 1 |
| 5 | Security Groups | 2 |
| 6 | NACL | 2 |
| 7 | SNS | 1 |
| 8 | Lambda Functions | 2 |
| 9 | CloudWatch Logs | 2 |
| 10 | VPCLink | 1 |
| 11 | AWS Gateway API | 1 |

AWS user account with appropriate roles for managing EC2 instances, AWS API Gateway, AWS Lambda, VPC, CloudWatch Logs etc.

**Step-by-Step Solution Implementation:**

1. Login / Sign into AWS Management Console
2. Create a VPC with couple of subnets. One subnet is configured as public subnet and other is configured as private subnet
3. Create an EC2 instance by selecting AMI from AWS marketplace. and attach this instance to private subnet. This instance hosts the AS400 API interfaces which in turn communicates with back-end AS400 servers through Infoview AS400 connector
4. Create a normal EC2 instance and attach this instance to public subnet. This instance will act as a bastion host or NAT Gateway host
5. Create a Network Load Balance and attach it to bastion host
6. Create a Lambda function i.e., InputTransformation and deploy (upload Jar), which transforms raw input json payload to as/400 compatible format and invokes the Program call API with this converted payload
7. Create AWS Gateway API and import the swagger collection which represents all the AS400 API Interfaces.
8. Create a VPC link and attach this to NLB
9. Map API Gateway Interfaces with Service API interfaces and Lambda functions using HTTP and VPCLink.

1. Login / Sig into AWS Management Console
2. Choose region from picklist as US East (Ohio)us-east-2
3. On successful sign in, Search and click on Gateway API from AWS services dashboard, shows Gateway API dashboard
4. Create new API by clicking on API link on the left side menu, leave some name and continue
5. Resource Importing – Swagger collection into API

1. Click on Resources \&gt; Actions picklist, then click on Import API under API Actions
2. Next copy and paste the Swagger collections into the text area, and then click on import
3. If everything seems to be ok then we can the below image

![](RackMultipart20220423-1-c0u157_html_465fd8013ab5fa11.png)

1. The following table depicts Gateway API interface mapping to AS400 Connector Service API which is hosted onto a EC2 private instance

| API Interface Name | Integration Type | Use Proxy Integration | Method | VPC Link | Endpoint URL|
| --- | --- | --- | --- | --- | --- |
| /connections | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections](http://x.x.x.xx:8080/connections) |
| /connections/{connection-name} | VPC Link | Keep it deselected | DELETE | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | VPC Link | Keep it deselected | GET | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | VPC Link | Keep it deselected | PUT | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name}/command | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/command-call](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/command-call) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | VPC Link | Keep it deselected | GET | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/invoke-program-call/{library-name}/{program-name} | Lambda | -na- | POST | InputTransformation | Lambda Region: us-east-2Name of the Lambda function: InputTransformation |
| connections/{connection-name}/program-call/{library-name}/{program-name} | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink | [http://x.x.x.xx:8080/connections/{connection-name}/program-call/{library-name}/{program-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/program-call/%7Blibrary-name%7D/%7Bprogram-name%7D) |
| /connections/{connection-name}/close | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink |
[http://x.x.x.xx:8080/connections/{connection-name}/close](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D/close) |
| /connections/{connection-name}/reopen | VPC Link | Keep it deselected | POST | Provide VPC link name. In the current implementation, it named as as400-vpclink |
[http://x.x.x.xx:8080/connections/{connection-name}/reopen](http://x.x.x.xx:8080/connections/%7Bconnection-name%7D/reopen) |

**Note:** Here x.x.x.xx is the IP address of the EC2 private instance, 8080 is the port on which AS400 Connector Service API is running.

1. The following table depicts Gateway API interface mapping to AS400 Connector Service API which is hosted onto a EC2 public instance

| API Interface Name | Integration Type | Use Proxy Integration | Method | Endpoint URL|
| --- | --- | --- | --- | --- |
| /connections | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections](http://x.x.x.xxx:8080/connections) |
| /connections/{connection-name} | HTTP | Keep it deselected | DELETE | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | HTTP | Keep it deselected | GET | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name} | HTTP | Keep it deselected | PUT | [http://x.x.x.xxx:8080/connections/{connection-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D) |
| /connections/{connection-name}/command | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/command-call](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/command-call) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | HTTP | Keep it deselected | GET | [http://x.x.x.xxx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/data-queue/{library-name}/{data-queue-name} | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/data-queue/{library-name}/{data-queue-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/data-queue/%7Blibrary-name%7D/%7Bdata-queue-name%7D) |
| /connections/{connection-name}/invoke-program-call/{library-name}/{program-name} | Lambda | -na- | POST | Lambda Region: us-east-2Name of the Lambda function: InputTransformation |
| connections/{connection-name}/program-call/{library-name}/{program-name} | HTTP | Keep it deselected | POST | [http://x.x.x.xxx:8080/connections/{connection-name}/program-call/{library-name}/{program-name}](http://10.0.1.233:8080/connections/%7Bconnection-name%7D/program-call/%7Blibrary-name%7D/%7Bprogram-name%7D) |
| /connections/{connection-name}/close | HTTP | Keep it deselected | POST |
http:// x.x.x.xxx:8080/connections/{connection-name}/close |
| /connections/{connection-name}/reopen | HTTP | Keep it deselected | POST | http:// x.x.x.xxx:8080/connections/{connection-name}/reopen |

**Note:** Here x.x.x.xxx is the IP address of the EC2 public instance, 8080 is the port on which AS400 Connector Service API is running.

1. All the above Gateway API can be tested with in the configuration editor. Please refer this sample-test-case-payloads.txt for test payloads.
2. Once the mapping configuration is done, then Create a Test Stage environment where the API gets deployed.

1. Create an SNS topic for publishing the received DTAQ events from the DTAQ
2. Create a Lambda function i.e. DQSNSEventProcessor, deploy (upload Jar) and subscribe to SNS topic
3. Testing API Gateway Interfaces in two ways
  1. From within the Gateway API
  2. Externally

**Testing existing solution:**

The following text talks about the general approach to be followed for testing the existing solution.

1. Login / Sig into AWS Management Console
2. Choose region from picklist as US East (Ohio)us-east-2
3. **Managing EC2 Instances**
  1. On successful sign in, Search and click on EC2 instances from AWS services dashboard, shows EC2 services dashboard
  2. Click on Instances under instances from the left side menu
  3. Start the below instances one after the other through by selecting check mark next to instance name and then click on Instance Start from the Instance State pick list
    1. as400gateway – is an EC2 instance and acts as bastion / NAT gateway host, which is part of public subnet under VPC. By default, application starts running on port 8080.
    2. CommonAPIServer – is an Ec2 instance and hosts the AS400 connector API interfaces.
4. **Managing Service Configuration**
  1. Application properties can be changed from the file present in this location /opt/as400-common-api/config/application-dev.properties
5. **Managing AWS API Gateway Interface**
  1. Search and click on API Gateway from within the AWS console, API Gateway Dashboard gets displayed
  2. Click check box next to &quot;AS400 Common API&quot;, Shows API interfaces info in a tree structure format where lot of insights can be drawn in terms of API specification, configuration, and testing
  3. Currently API interfaces are deployed in the Test Stage environment. To get Test stage environment info, click on Stages in the left side menu. Here site URL info is available.

Ex. [https://46oht9t3f8.execute-api.us-east-2.amazonaws.com/test](https://46oht9t3f8.execute-api.us-east-2.amazonaws.com/test)

1. **Logs Verification**
  1. CommonAPIServer logs can be verified from this location /var/log/as400-common-api-service.log
  2. Lambda function logs
    1. Search and click on Lambda function from within the AWS console, Lambda dashboard gets displayed
    2. Click on Functions and can see the below available functions
      1. InputTransformation: Transforms Input json payload to as/400 compatible format
      2. DQSNSEventProcessor: AS/400 DTA Queue SNS Lambda Integration
    3. To view logs pertaining to Lambda \&gt; Functions \&gt; DQSNSEventProcessor
      1. Click on DQSNSEventProcessor, Displays dashboard with configuration, Permission and Monitoring
      2. Click on Monitoring tab, it shows Monitoring dashboard along with the CloudWatch metrics. Click on &quot;View logs in CloudWatch&quot;. This is the path where logs can be found CloudWatch \&gt; CloudWatch Logs \&gt;

Log groups \&gt; /aws/lambda/DQSNSEventProcessor

1. **Managing Application Service**

By using tool Putty SSH login to the public EC2 instance Ex.AS400Gateway and then use below command to login to private EC2 instance. **ssh ec2-user@x.x.x.xx**

where x.x.x.xx represents IP address of private EC2 instance where the actual service is hosted.

  1. To know the application status: sudo service as400-common-api-service status
  2. To stop the application: sudo service as400-common-api-service stop
  3. To start application: sudo service as400-common-api-service start

**License Management:**

The IBM i connector requires a license file &quot;as400-license.lic&quot; from Infoview to enable access to specific IBM i system(s).

Managing license in different ways by using different protocols such as S3, HTTP/HTTPS, FTP, FILE, SMB etc. and accessing it through these protocols in our application needs to configure required properties in **application-dev.properties** file.

Available Protocols to load license file/truststore file (HTTP,HTTPS, FTP, SMB, S3, FILE, CLASSPATH_)_

what protocol used to load license file/truststore file that need to be configured as below in application-dev.properties file as below.

licenseFileProtocol=S3

Truststore file is used to establish the secure connection with IBM i AS400 system. if secure connection property set as true then needs to configure truststore file protocol in application-dev.properties file as below.

truststoreFileProtocol=S3

Following table contains the properties related to protocols requires to be configure:

| **#** | **Protocol Name** | **Properties** |
| --- | --- | --- |
| 1 | S3 | s3.bucket=\&lt;path-to-bucket\&gt;<br>s3.region=us-east-2</br><br>s3.accessKey=ENC(\&lt;encrypted-access-key\&gt;)</br><br>s3.secretKey=ENC(\&lt;encrypted-secret-key\&gt;)|
| 2 | HTTP/HTTPS | http.url=\&lt;url-URL\&gt;<br>http.dir.path=\&lt;license-file-path\&gt;</br><br>http.username=ENC(\&lt;encrypted-user-name\&gt;)</br>http.password=ENC(\&lt;encrypted-pwd\&gt;)
 |
| 3 | FTP | ftp.host=\&lt;ftp-host\&gt;<br>ftp.dir.path=\&lt;path\&gt;</br><br>ftp.username=ENC(\&lt;encrypted-user-name\&gt;)</br><br>ftp.password=ENC(\&lt;encrypted-pwd\&gt; ) </br>|
| 4 | FILE/SMB | file.Path=\&lt;path-to-license-file\&gt;
 |
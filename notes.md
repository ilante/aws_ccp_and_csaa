# Notes from AWS ccp and csaa

From Udemy: 
* [ccp (Ultimate AWS Certified Cloud Practitioner - 2022)](https://www.udemy.com/course/aws-certified-cloud-practitioner-new) 
* [saa-c03 (Ultimate AWS Certified Solutions Architect Associate SAA-C03)](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03)


* [Find out](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/?p=ngi&loc=4) if your service is available in your Region
* Customer responsibility *in* the cloud
* AWS responsible for security *of* the cloud

Acceptable Use Policy
* No illegal, harmful or offensive use or content
* No security violations
* No network abuse
* No email or other message abuse 

[Flashcards](https://flashcards.io/app?url=https://quizlet.com/413033886/aws-certified-cloud-practitioner-flash-cards/&mode=play)

## AWS CLI
```
aws configure
AWS Access Key ID [None]: <insert> 
AWS Secret Access Key [None]: <insert>
Default region name [None]: eu-central-1
Default output format [None]: <press-enter> 
```
`aws iam list-users`

## Cloud shell

* Since you're logged in already no keys required
* When running aws iam list-users` from the cloud shell
	* This is going to return an API call as if the credentials where being used
			* API - Application Programming Interface; set of protocols, procedures, and tools that allow interaction between 2 applications. It's a sw intermediary that delivers a request to the server and then relays a response back to the client. Avoid reinventing the wheel...

* You can create persisting files in CloudShell
* Possibility to download and upload files

Your home:
```
/home/cloudshell-user
```
* Obv you can open new tabs and all the other stuff too...

## IAM Roles for Services

* Services can perform actions on your behalf &rarr; use IAM roles to assign permissions

### IAM Security Tools

* IAM Credentials Repoert = account level
	* Can be used for auditing
* IAM Access Advisor = user-level
* Assign users to groups!
* Assign permissions at group level

# Shared Responsibility model

Your responsibility:

1. Management and monitoring of: Users, Groups, Roles, Policies; 
2. Enabling MFA on all accounts
3. Rotating all keys often
4. Use IAM tools to apply appropriate permissions
5. Analyse access patterns & and review permissions

## IAM summary

* Users: physical users, have pw for console
* Groups: contain users only
* Policies JSON doc outlining permissions for users or groups 
* Roles: for EC2 instances or AWS services
* Security: set up MFA & strong pw policy
* AWS CLLI: manage services 
* AWS SDK: managing services using a programming languageA
* Access Keys: for CLI or SDK
* Def IAM entity; defines a set of permisssions for making aws service requests, that will be used by aws services
	* Eg. EC2 Instance that has a Role assigned
	* The Role inturn has a set of permissions
	* That way the EC2 instance can perform actions on my behalf
* The iam credentials report is an iam security tool
* The iam access advisor is a security tool as well

# EC2

* `pbcopy` pasteboard copy &rarr;
* 7 types of EC2 instances
* Eg. m5.2xlarge
	* m: instance class
	* 5: generation
	* 2xlarge: size of the instance class

## Security Groups

* "Firewall" control in and outbound traffic
* Regulate:
	* Access ports
	* Authorised IP ranges IPv4 and IPv6
* Ports:
	* 22 &rarr; ssh
	* 21 &rarr; FTP file transfer protocol, to upload files inot a file share
	* 22 &rarr; SFTP secure file transfer protocol - for uploading files using ssh
	* 80 &rarr; HTTP accessing unsecured websites
	* 443 &rarr; HTTPS accessing secured websites
	* 3389 &rarr; RDP remote desktop protocol, to log into a win instance
* You can attach several security groups to one instance

## SSH

CLI utilty for win < 10 putty.
* `ssh -i <path_to_key> ec2-user@<public_ipv4>`
* `chmod 0400` to change permissions to only user read only
* sometimes you should add another ssh group that allows `Source` "Anywhere-IPv6" instead of "Custom"

## EC2 Instance Connect

* Browserbased ssh session

## 11. Beanstalk Hands On

Why are the s3 buckets not deleted upon deleting the beanstalk application?

* Elastic Beanstalk deletes most related objects 
* Bucket policy protects from accidental deletion. So we need to delete the policy first. [See](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.S3.html#AWSHowTo.S3.delete-bucket)
* Policy is in the *Permissions* section of the bucket porperties in the S3 console

```
 {
            "Sid": "eb-58950a8c-feb6-11e2-89e0-0800277d041b",
            "Effect": "Deny",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:DeleteBucket",
            "Resource": "arn:aws:s3:::elasticbeanstalk-us-east-1-186290353868"
        }
    ]
}
```
* Delete entire bucket policy

# Ch 13 ccp 

## aws Simple Queue Service - SQS

* Usdd to *decouple* applications
* SQS standard queue
* Fully managed
* 1 Message up to 10 000's p/sec
* Delete messages upon cosumer reading them
* Low latency
* independent scaling

## Simple Notification Service

* SNS is a durable, secure, fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and and serverless applications. It uses a push based system

## Kinesis

* Real time big data streaming
* At any scale
* Low latency
	* Kinesis Data Streams
	* Kinesis Data Firehose: S3, Redshift, ElasticSearch
	* Kinesis Data Analytics
	* Kinesis Video Streams

Decoupling your applications is an important principle that is exploited by SNS and SQS.

# Ch 17 saa - Decoupling applications SQS, SNS, Kinesis, Active MQ

* Decoupling is useful to avoid problems arising bia spikes
* Can be done by 
	* SQS queue model
	* SNS pub/sub model
	* Kinesis real time streaming model

## SQS 

* Producers send messages to the Q
* Consumers poll messages from the Q and will get information
* The message is deleted from the Q
* SQS works as a buffer between Producer and Consumer

## SQS Standard Q

* 10 y old
* Fully managed
* Attributes: 
	* Unlimited throughput, unlimited number of messages in queue
	* Default retention of messages: 4 d - max 14 d
	* LOW latecy: < 10 ms on publish and receive
	* 256 kb max (2^8)

* Messages are sent via SDK the API that is used is called "SendMessage"
* Messages persists in SQS until a consumer deletes it
* E.g. processing an order
	* Order id
	* Customer id
	* Any attributes you can imagine
* Unlimited throughput

## SQS Consumers - Consuming Messages

* Consumers could be running on EC2, servers, AWS Lambda
* Consumer polls for polls for messages and can receive up to 10 messages at a time
	* "Do you have any msg?"
	* Reads msg
	* Deletes msg via "delete message API" &rarr; avoiding that any other consumer can see the msg
* Consumers receive and process messages in parallel
* At least once delivery
* Best-effort message ordering
* Consumers delete messages after processing
* Able to scale consumers horizontallly to improve throughput of processing
* Eg. Autoscaling triggered by CloudWatch Metric: q length (Alarm)

## SQS Security

* Encryption
	* In-flight encrytption using HTTPS API
	* At rest: KMS
	* Client-side encryption, if client wants to perform encryption/decryption itself

* IAM policies
* SQS Acess Policies

* Message retention: decides how long the message will be held in the queue before being dropped

## SQS 

* Message visibility timeout: avoiding that message is read by other consumers - default 30 sec
* SQS dead letter queue (DLQ)
	* For consumer failing to process a message within v timeout
	* Message goes back to queue
	* Upon exceeding the MaximumReceives threshold message is moved to DLQ
	* Useful for debugging
		* Expiration time for DLQ ~ 14d
* Long polling 
	* Decreases number of api calls made to SQS &rarr; efficiency ++; latency ++
	* LP preferable to short p

* SQS - Request-Response Systems
	* Implemented by use of SQS Temporary Queue Client
	* Leverages virtual queues instead of creating/deleting SQS queues &rarr; cost

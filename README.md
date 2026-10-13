<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# VPC Endpoints

**Project Link:** [View Project](https://nextwork.ai/projects/749fab76-0a0b-5822-9b31-c7863050c9bd)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_09bcaa8a)

## Introducing Today's Project!

### What is Amazon VPC?

Amazon VPC is a service that lets you provision a logically isolated section of the Amazon Web Services cloud where you can launch and manage your AWS resources in a virtual network that you fully control, including your own IP address ranges, subnets, route tables, and network gateways. It is useful because it gives you advanced security and granular control over your cloud architecture, allowing you to isolate sensitive compute instances and databases from the public internet, configure custom network boundaries, and establish secure hybrid or private connectivity to other AWS services and on-premises data centers.

### How I used Amazon VPC in this project

In today's project, I used Amazon VPC to build a secure network environment, launch an EC2 instance in a public subnet, and configure an Amazon S3 Gateway endpoint that allows my virtual machine to communicate privately and securely with my S3 storage bucket without routing traffic through the public internet.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how instantly saving a strict VPC endpoint bucket policy would lock me out of the AWS Management Console and trigger access denied warnings across my browser session, highlighting just how powerful resource-based policies are for enforcing network boundaries.

### This project took me...

This project took me 40 minutes only

## In the first part of my project...

### Step 1 - Architecture set up

In this step, I will set up our core cloud architecture by creating a custom VPC, launching an EC2 instance in a public subnet, and creating an S3 bucket with test files because these foundational resources are required to test and demonstrate private VPC endpoint connectivity later

### Step 2 - Connect to EC2 instance

In this step, I will connect directly to my EC2 instance using EC2 Instance Connect because it allows me to access the instance's terminal to run AWS CLI commands and interact with my AWS environment.

### Step 3 - Set up access keys

In this step, I will create an IAM access key because my EC2 instance needs authentication credentials to make programmatic requests and interact with AWS services via the CLI.

### Step 4 - Interact with S3 bucket

In this step, I will connect to my S3 bucket and run AWS CLI commands from my EC2 instance terminal because I need to test whether my instance can successfully interact with my S3 storage and upload test files.

## Architecture set up

I started my project by launching a custom VPC, an Amazon EC2 instance in a public subnet, and an S3 bucket with uploaded test files to build the core architecture needed for testing VPC endpoint connectivity.

I also set up an S3 bucket in the same region with uploaded test files to provide the storage target needed for testing connectivity and bucket policies.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_4334d777)

## Access keys

### Credentials

To set up my EC2 instance to interact with my AWS environment, I configured my unique AWS Access Key ID and AWS Secret Access Key to authenticate programmatic requests securely through the command line. These credentials act as a digital username and password pair, granting my virtual machine permissions to communicate directly with AWS services. Additionally, I defined my default region name to ensure that all AWS CLI commands target the correct geographical data center where my resources reside. Finally, I left the default output format option blank to rely on the standard JSON structure, which formats all command line responses cleanly for easy reading. Together, these four configuration parameters establish a fully authenticated and operational communication bridge between my compute instance and the broader AWS ecosystem, enabling seamless administrative control.

Access keys are long-term credentials consisting of an Access Key ID and a Secret Access Key used to authenticate programmatic requests made to AWS via the CLI, SDKs, or APIs.

Secret access keys are confidential companion credentials to your access key ID that function similarly to a password, acting as a cryptographic signature to authenticate programmatic requests made to AWS. When interacting with AWS through the Command Line Interface, SDKs, or APIs, these keys verify your identity and authorize actions against your account resources.

Because they grant direct, unmonitored access to your AWS environment, secret access keys must be guarded closely. If exposed or compromised, unauthorized individuals could leverage them to provision resources, view sensitive data, or incur unwanted charges. This is why AWS recommends treating them with the same security measures as a master password, restricting their distribution, rotating them regularly, and deleting them immediately when they are no longer required.

As a security best practice, long-term access keys should be avoided in production environments whenever possible. Instead, developers and system administrators should rely on temporary security credentials, such as IAM roles attached directly to AWS compute resources like EC2 instances. These roles handle credential generation and rotation automatically behind the scenes, eliminating the risks associated with hardcoding credentials or storing them in local files.

### Best practice

The best practice alternative to using long-term access keys is to use IAM roles.

When you attach an IAM role to an EC2 instance, the instance automatically inherits temporary credentials managed securely by AWS, eliminating the need to hardcode or manually manage keys.

## Connecting to my S3 bucket

The command I ran was aws s3 ls. This command is used to list all the S3 buckets available in your AWS account.

The terminal responded with 2026-10-13 02:28:10 nextwork-vpc-project-jasahorro. This indicated that the access keys I set up successfully authenticated my EC2 instance and allowed it to securely communicate with AWS to list the S3 buckets in my account.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_4334d778)

## Connecting to my S3 bucket

I also tested the command aws s3 ls s3://nextwork-vpc-project-jasahorro, which returned the list of files stored inside my bucket (NextWork - Denzel is awesome.png, NextWork - Lelo is awesome.png, and test.txt)

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_4334d779)

## Uploading objects to S3

To upload a new file to my bucket, I first ran the command sudo touch /tmp/nextwork.txt to create an empty text file in the /tmp/ directory of my EC2 instance, followed by aws s3 cp /tmp/nextwork.txt s3://nextwork-vpc-project-jasahorro to upload it into my S3 bucket. This command creates a file copy directly from my EC2 instance storage and transfers it to the cloud storage bucket.

The second command I ran was aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-jasahorro. This command will copy the local text file you created in your EC2 instance (/tmp/test.txt) and upload it directly into your Amazon S3 bucket.

The third command I ran was aws s3 ls s3://nextwork-vpc-project-jasahorro, which validated that the new test.txt file was successfully uploaded to my S3 bucket and is now stored alongside my other files.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_3e1e79a2)

## In the second part of my project...

### Step 5 - Set up a Gateway

In this step, I will create a VPC endpoint (specifically an S3 gateway endpoint) because it provides a private, secure connection between my VPC and Amazon S3 without routing traffic through the public internet.

### Step 6 - Bucket policies

In this step, I will create a restrictive bucket policy for my Amazon S3 bucket to deny all public traffic and external access, ensuring that only my newly created VPC endpoint can connect to and interact with the bucket.

### Step 7 - Update route tables

In this step, I will update my public subnet route table to include a route pointing to my VPC endpoint because I need to ensure that traffic bound for Amazon S3 is routed privately through the endpoint rather than via the public internet.

### Step 8 - Validate endpoint conection

In this step, I will validate my VPC endpoint connection because I need to confirm that my EC2 instance can successfully access and interact with my S3 bucket privately through the newly configured endpoint and route table.

## Setting up a Gateway

I set up an S3 Gateway, which is a specific type of VPC endpoint used for Amazon S3 and DynamoDB that adds a route to your VPC route table, directing traffic bound for S3 straight to the gateway instead of routing it through the public internet.

### What are endpoints?

An endpoint is a service that allows private connections between your VPC and other AWS services without needing the traffic to go over the public internet, ensuring your data stays secure within the AWS network.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_09bcaa8a)

## Bucket policies

A bucket policy is a resource-based AWS IAM policy written in JSON format that allows or denies permissions to your Amazon S3 bucket and the objects stored within it, giving you centralized control over who can access your data and what specific actions they are allowed to perform.

My bucket policy will deny all public traffic and external access to my Amazon S3 bucket, ensuring that only requests originating from my specific VPC endpoint ID (vpce-0e4528d741a4050a4) are permitted to interact with the storage bucket.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_7316a13d)

## Bucket policies

Right after saving my bucket policy, my S3 bucket page showed 'denied access' warnings. This was because my new bucket policy explicitly denies all actions to everyone—including my browser console credentials—unless the request originates from my specific VPC endpoint (vpce-0e4528d741a4050a4). Since the AWS Management Console accesses the bucket over the public internet rather than through the VPC endpoint, Amazon S3 instantly blocked the connection and displayed access denied warnings.

I also had to update my route table because my public subnet's route table did not yet have a rule directing traffic bound for Amazon S3 to our VPC endpoint, meaning requests from my EC2 instance were still attempting to traverse the public internet instead of utilizing the private endpoint route.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_4ec7821f)

## Route table updates

To update my route table, I navigated to the Endpoints console, selected my VPC endpoint, chose Manage route tables, and checked the box next to my public route table (rtb-05ca9a5d4445fd7a7) before saving the modification. This automatically added the required destination route (pl-63a5400a) pointing to my VPC endpoint (vpce-0e4528d741a4050a4).

After updating my public subnet's route table, my terminal successfully returned the list of objects stored inside my Amazon S3 bucket (NextWork - Denzel is awesome.png, NextWork - Lelo is awesome.png, and test.txt). This proved that my EC2 instance regained access to the storage bucket privately through the new VPC gateway endpoint and route table, successfully bypassing the internet-blocking security policy.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_d116818e)

## Endpoint policies

An endpoint policy is an IAM resource-based policy attached directly to a VPC endpoint that controls which AWS services, accounts, or specific actions are allowed or denied through that endpoint, giving you granular security control over traffic flowing from your VPC.

I updated my endpoint's policy by changing the Effect from Allow to Deny, which instantly blocked all access through the VPC endpoint. I could see the effect of this right away, because when I ran the aws s3 ls command again in my EC2 terminal, it immediately returned an Access Denied error, proving that the endpoint policy successfully cut off communication to the S3 bucket.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/749fab76-0a0b-5822-9b31-c7863050c9bd_3e1e79a3)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/749fab76-0a0b-5822-9b31-c7863050c9bd)*

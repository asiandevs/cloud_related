# Install and Configure AWS CLI on CentOS
The AWS Command Line Interface (AWS CLI) allows you to interact with various AWS services from the command line. Here's how to install and configure it on Centos/AWS Linux.

## Step 1: Download the AWS CLI Installation Package

Use the following `curl` command to download the AWS CLI installation package:

```bash
# uname -a
Linux ip-10-0-14-162.ec2.internal 6.1.56-82.125.amzn2023.x86_64 #1 SMP PREEMPT_DYNAMIC Tue Oct 10 17:03:53 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

## Step 2: Install AWS CLI

Unzip the downloaded installer file using the following command:

```bash
unzip awscliv2.zip
```

## Step 3: Install AWS CLI

Install AWS CLI using the following command:

```bash
# ./aws/install
You can now run: /usr/local/bin/aws --version
```

After installation, you can run the AWS CLI to verify its version:

```bash
/usr/local/bin/aws --version
```

You should see output similar to this:

```
aws-cli/2.9.19 Python/3.9.16 Linux/6.1.56-82.125.amzn2023.x86_64 source/x86_64.amzn.2023 prompt/off
```

## Step 4: Configure AWS CLI

To configure the AWS CLI, use the following command. You will be prompted to provide your Access Key ID, Secret Access Key, and other configuration details:

```bash
/usr/local/bin/aws configure
```

You'll be prompted to enter your AWS Access Key ID, AWS Secret Access Key, default region name, and output format.

- **Access Key ID and Secret Access Key:** These are the credentials you obtained from AWS IAM.
- **Default Region Name:** This is the AWS region you want to use by default (e.g., `us-east-1`, `us-west-2`, etc.).
- **Default Output Format:** You can leave this as "json" for machine-readable output.

## To get the Security Credential 
1. Log into AWS console and search with IAM. Select IAM to get into the IAM console.
![image](https://github.com/asiandevs/cloud_services/assets/37457408/a85090fe-cc31-4d04-8887-f56bf8698807)

2. Select 'My security credentials' option available in the right to create and manage your security credentials.
3. Click on 'Access Keys' drop down box and click 'Create New Access Key' option.
4. Once you click the 'Create New Access Key' option, the key will be created. Download the key to configure AWS CLI in Linux machine. You can view the key by checking 'Show Access key'.
   ![image](https://github.com/asiandevs/cloud_services/assets/37457408/dfb6ab34-7489-470e-933f-4c8e21802549)

After entering these details, the AWS CLI will store them in a configuration file (usually `~/.aws/credentials` and `~/.aws/config`).

## Step 5: Verify AWS CLI Installation
To verify that the AWS CLI is set up correctly, you can run a simple command like `aws s3 ls`, which lists your S3 buckets if you have any. This command will help you confirm that the AWS CLI is configured and functioning as expected.

```
# aws s3 ls
2023-04-08 22:18:20 amitybkt01
2023-05-11 11:02:01 cf-templates-11p6u6ki0e4ec-us-east-1
```
Your AWS CLI is now set up and ready to use. You can start using AWS services and manage your resources from the command line. If you want to explore various AWS CLI commands and their usage, you can refer to the official [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/index.html).

After completing these steps, your AWS CLI is configured and ready to use on your CentOS system.

Now you can start interacting with AWS services from your command line.
```


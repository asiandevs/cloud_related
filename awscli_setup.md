# Install and Configure AWS CLI on CentOS
The AWS Command Line Interface (AWS CLI) allows you to interact with various AWS services from the command line. Here's how to install and configure it on Centos/Linux.

## Step 1: Download the AWS CLI Installation Package

Use the following `curl` command to download the AWS CLI installation package:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

## Step 2: Install AWS CLI

Unzip the downloaded installer file using the following command:

```bash
unzip awscliv2.zip
```

## Step 3: Install AWS CLI

Install AWS CLI using the following command:

```bash
./aws/install
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

After entering these details, the AWS CLI will store them in a configuration file (usually `~/.aws/credentials` and `~/.aws/config`).

3. **Verify AWS CLI Installation:**
To verify that the AWS CLI is set up correctly, you can run a simple command like `aws s3 ls`, which lists your S3 buckets if you have any. This command will help you confirm that the AWS CLI is configured and functioning as expected.

Your AWS CLI is now set up and ready to use. You can start using AWS services and manage your resources from the command line. If you want to explore various AWS CLI commands and their usage, you can refer to the official [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/index.html).

After completing these steps, your AWS CLI is configured and ready to use on your CentOS system.

Now you can start interacting with AWS services from your command line.
```


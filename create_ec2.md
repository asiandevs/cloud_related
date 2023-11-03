# Step-by-Step Guide: Launching an AWS EC2 Instance

1. **Sign in to AWS Console:**
   Log in to your AWS account at [https://aws.amazon.com/](https://aws.amazon.com/).

2. **Access the EC2 Dashboard:**
   After logging in, go to the AWS Management Console, and in the "Services" dropdown, select "EC2" under "Compute."

3. **Launch an Instance:**
   To launch an EC2 instance, follow these steps:

   - Click the "Instances" link on the EC2 Dashboard.
   - Click the "Launch Instance" button.

4. **Choose an Amazon Machine Image (AMI):**
   An AMI is a pre-configured virtual machine. You can choose an AMI based on your needs, such as Amazon Linux, Ubuntu, Windows, etc.

5. **Choose an Instance Type:**
   Select the hardware specifications of your EC2 instance, such as the number of vCPUs, memory, and GPU options. You can also use the "Free tier only" checkbox to filter for eligible instances if you're eligible for AWS's free tier.

6. **Configure Instance Details:**
   Set options like the number of instances to launch, network settings (Virtual Private Cloud - VPC), IAM roles, and user data scripts if needed.

7. **Add Storage:**
   You can specify the size and type of the root volume and attach additional EBS (Elastic Block Store) volumes to your instance if necessary.

8. **Add Tags (Optional):**
   Tags are key-value pairs that you can use to categorize and identify your instances. This step is optional but can be helpful for organization.

9. **Configure Security Group:** [ Allow access from known IP addresses only.]
   A security group acts as a firewall for your instance. Configure inbound and outbound rules to control network traffic. Be sure to allow SSH (port 22) or other protocols you need.
NOTE: In the Security Group enable access to following ports from anywhere Here only for learning purpose| Rules with source of 0.0.0.0/0 or ::/0 allow all IP addresses to access your instance.

![image](https://github.com/asiandevs/cloud_services/assets/37457408/f0007633-7f0c-4908-b1c2-654157a8420e)


11. **Review and Launch:**
    Review your configuration settings, and then click "Launch" when you're ready to proceed.

12. **Create or Choose a Key Pair:**
    If you don't have an existing key pair, you'll need to create one. This is crucial for SSH access to your instance. Download and securely store the private key (.pem) file.

13. **Launch Instances:**
    After creating or selecting a key pair, click "Launch Instances."

14. **View Instances:**
    You'll be directed to the Instances view, where you can see the status of your new instance. It may take a few minutes for it to become available.

15. **Connect to Your Instance:**
    Once the instance is running, you can connect to it using SSH (for Linux) or RDP (for Windows). Use the key pair you created or selected in step 11.

That's a basic overview of launching an EC2 instance. Remember to secure your instance, manage your security group rules, and regularly back up your data. Also, be mindful of costs, as AWS charges based on usage, so stop or terminate instances when you're not using them to avoid unnecessary expenses.

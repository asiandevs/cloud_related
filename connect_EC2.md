### connecting to an EC2 Linux instance

**Prerequisites:**

1. **Amazon Web Services (AWS) Account:** You need to have an AWS account to create and manage EC2 instances.

2. **EC2 Instance:** Make sure you have launched an EC2 instance. You'll need the instance's public IP address or DNS name to connect. Also, ensure that the security group associated with the instance allows SSH traffic (port 22) from your IP address.

3. **Key Pair:** When you launch an EC2 instance, you should select or create an SSH key pair. You'll need the private key file (.pem) for this key pair to connect to your instance. Store this key securely.

**Connecting to your EC2 Linux instance:**

Option 1: From the Comnsole 

![image](https://github.com/asiandevs/cloud_services/assets/37457408/324c96e7-83b6-43bf-8d17-c91f0dc91dd7)

![image](https://github.com/asiandevs/cloud_services/assets/37457408/7ace86b0-787d-48f8-9da4-e55842626c5a)

![image](https://github.com/asiandevs/cloud_services/assets/37457408/cc51dcac-5cbe-4ed3-a032-1f25df555602)


Option 2: From Putty 

1. **Get the Public IP address from EC2:**
![image](https://github.com/asiandevs/cloud_services/assets/37457408/f6c74ed3-4d31-45e7-baa3-68bb0d901409)

3. **Open a putty Terminal:** use the public address and user name. For AWS EC2 - ec2-user
  ![image](https://github.com/asiandevs/cloud_services/assets/37457408/82199834-098f-4709-a860-d7a23f54eab5)

4. Define the private key path [ which yuo define during provision of EC2]
   ![image](https://github.com/asiandevs/cloud_services/assets/37457408/deade413-6948-475b-a13a-cdfa77c60850)
5. When you done, click open

6. Accept and you are in
   ![image](https://github.com/asiandevs/cloud_services/assets/37457408/7cdc35ab-8c30-4556-9d04-8e1b8a55ce85)

Option 3: From a Windows Terminal

1. **Open a Terminal:** On your local machine (PC or Mac), open a terminal window.

2. **Change Directory:** Use the `cd` command to navigate to the directory where your private key file is located.

3. **Change Key Permissions:** To ensure the private key file is not accessible to others, you need to restrict its permissions. Use the `chmod` command to set the appropriate permissions:

   ```
   chmod 400 your-key.pem
   ```

4. **Connect to the EC2 instance:** Use the `ssh` command to connect to your EC2 instance. Replace `your-key.pem` with the actual private key file, and replace `your-ec2-public-ip` with the public IP address or DNS name of your EC2 instance:

   ```shell
   ssh -i your-key.pem ec2-user@your-ec2-public-ip
   ```

   If you're using a different Linux distribution, replace `ec2-user` with the appropriate username (e.g., `ubuntu` for Ubuntu instances).

5. **Accept SSH Host Key:** When connecting for the first time, you may see a message asking if you want to continue connecting. Type "yes" to accept the EC2 instance's SSH host key.

6. **You're In:** If the private key and permissions are set correctly, you should now be logged into your EC2 instance via SSH. You'll see a command prompt for the remote instance, and you can start running commands.

Remember to disconnect from the instance when you're done. You can do this by simply typing `exit` and pressing Enter in the terminal.

That's it! You've successfully connected to your EC2 Linux instance. You can now manage your instance, install software, and perform other tasks as needed.

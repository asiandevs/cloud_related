# Mounting Amazon EFS on an EC2 Instance

## Step 1: Create an EFS File System
1. Navigate to the **AWS Console** > **EFS**
2. Click **Create File System**

## Step 2: Launch an EC2 Instance
- Create an EC2 instance using **Amazon Linux 2** as the AMI.

## Step 3: Configure EFS Security Group
1. Go back to **EFS** and select your file system
2. Under **Network**, note the EFS details
3. **Security Group Configuration**:
   - Go to your **Security Groups**
   - Select the security group associated with your EFS
   - Edit **inbound rules** → **Add rule**:
     - Type: `NFS`
     - Port Range: `2049`
     - Source: Your EC2 instance's security group

## Step 4: Connect EC2 Instance to EFS
1. SSH into your EC2 instance
2. Install the EFS utilities:
   ```bash
   sudo yum install -y amazon-efs-utils

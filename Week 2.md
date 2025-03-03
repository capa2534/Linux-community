### SSH (Secure Shell) connections, EC2 instance management, and SCP (Secure Copy Protocol) 

1 Created an EC2 Instance in AWS:

- Chose an Amazon Machine Image (AMI), probably a Linux-based system (e.g., Amazon Linux, Ubuntu, or RHEL).
- Selected an instance type (e.g., t2.micro for free tier usage).
- Configured network settings (e.g., ensuring the correct inbound rules for SSH access).
- Created or used an existing SSH key pair.
- Launched the instance.
                                 
2 Checked the Public IP Address
- Found the Public IPv4 address or Public DNS under the EC2 instance details.

3 Connecting to the EC2 Instance via SSH

- Once the instance was running, you connected to it from your virtual machine using SSH. The command would have been:

`ssh -i my-key.pem ec2-user@<PUBLIC_IP>`

or, for Ubuntu-based instances:

`ssh -i my-key.pem ubuntu@<PUBLIC_IP>`

The .pem key file must have proper permissions: chmod 400 my-key.pem


Transferring Files with SCP

After successfully connecting via SSH, you attempted to transfer files using SCP (Secure Copy Protocol). SCP allows copying files between your local machine (virtual machine) and the EC2 instance over SSH.
Uploading a File to EC2

From your virtual machine to the EC2 instance:

scp -i my-key.pem file.txt ec2-user@<PUBLIC_IP>:/home/ec2-user/

This command:

    Uses the -i my-key.pem key file for authentication.
    Transfers file.txt to /home/ec2-user/ on the remote EC2 instance.

Downloading a File from EC2

From the EC2 instance to your virtual machine:

scp -i my-key.pem ec2-user@<PUBLIC_IP>:/home/ec2-user/file.txt .

This command downloads file.txt from the remote instance to your current local directory.

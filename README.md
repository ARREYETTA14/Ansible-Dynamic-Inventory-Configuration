# Ansible-Dynamic-Inventory-Configuration
Dynamic Inventory Configuration in AWS

**1. Create Instances:**
- Launch three instances: one named “Ansible-Master,” the second “Worker-node-2,” and the third “Worker-node-3.”
- Add additional tags to the worker nodes as Env: Dev.
  **Note:** Ensure the Security Group (SG) of the worker nodes allows TCP traffic from the SG of the Ansible-Master node.

![image](https://github.com/user-attachments/assets/4cf67fb4-5e01-4b05-9d4d-9fc7af5665bb)

**2. Assign Permissions:**
Give the Ansible-Master node the 'EC2FullAccess' permission.

**3. SSH into Ansible-Master:**
- Update the server:
```bash
sudo yum update -y
```
- Check the version of Python. If it is not version 3 or above, run the following command to install it:
```bash
python3 --version
sudo yum install python3 -y
```
- Install PIP:
```bash
sudo yum install python3-pip -y
```
**4. Create Directory for Inventory:**
- Create a directory where the inventory file will reside:
```bash
sudo mkdir -p /opt/ansible/inventory
cd /opt/ansible/inventory
```
**5. Create the aws_ec2.yaml File:**
- In this directory, create the aws_ec2.yaml file to filter required instances for task execution:
```bash
sudo nano aws_ec2.yaml
```
**6. Add the Following Code to aws_ec2.yaml:**
```yaml
---
plugin: amazon.aws.aws_ec2
regions:
  - sa-east-1
keyed_groups:
  # Create groups based on the value of the 'Env' tag.
  - key: tags.Env
    prefix: tag_Env_
    separator: ""
filters:
  # Filter instances that have the 'Env' tag (prod, dev, etc.).
  tag:Env:
    - 'Dev' 
hostnames:
  # Define the hostname precedence (use instance's private IP or tag name as hostname).
  - tag:Name
  - private-ip-address
compose:
  # Set ansible_host to use the private IP for SSH connections.
  ansible_host: private_ip_address
```
**7. Go Back to Home Directory:**
```bash
cd
```
**8. Install Boto and Boto3:**
```bash
pip3 install boto3 -y
pip3 install botocore
```
**9. Install the Amazon AWS Collection:**
```bash
ansible-galaxy collection install amazon.aws
```
**10. Install Ansible:**
```bash
sudo amazon-linux-extras install ansible2 -y
```
**11. Test AWS Environment Access:**
```bash
ansible-inventory -i /opt/ansible/inventory/aws_ec2.yaml --list
```


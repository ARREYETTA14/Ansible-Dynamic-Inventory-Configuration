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
``bash
sudo yum install python3-pip -y
```



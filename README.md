# Ansible-Dynamic-Inventory-Configuration
Dynamic Inventory Configuration in AWS *(3 Amazon Linux 2023 instances: 1 Master Node and 2 Worker Nodes)*

**1. Create Instances:**
- **Ansible-Master Security Group**: Allow inbound **SSH (Port 22)** from your IP address.
- **Worker Nodes Security Group**: Allow inbound **SSH (Port 22)** from the Ansible-Master Security Group, and allow inbound **HTTP (Port 80)** from 0.0.0.0/0 so you can view the website in your browser later.
- Launch three instances: one named “Ansible-Master,” the second “Worker-node-2,” and the third “Worker-node-3.”
- Add additional tags to the worker nodes as ```Env: Dev```.
  **Note:** Ensure the Security Group (SG) of the worker nodes allows TCP traffic from the SG of the Ansible-Master node.

![image](https://github.com/user-attachments/assets/4cf67fb4-5e01-4b05-9d4d-9fc7af5665bb)

**2. Assign IAM Permissions to Master Node:**
- In the AWS IAM Console, create an IAM Role for **EC2** and attach the ``AmazonEC2ReadOnlyAccess`` policy (or ``AmazonEC2FullAccess`` if you plan to create/terminate instances via Ansible later).
- Attach this IAM Role to your Ansible-Master EC2 instance

**3. SSH into Ansible-Master:**
- Update the server:
```bash
sudo dnf update -y
```
- Check the version of Python. If it is not version 3 or above, run the following command to install it:
```bash
python3 --version
sudo dnf install python3 -y
```
- Install PIP:
```bash
sudo dnf install python3-pip -y
```

**4. Install Ansible, Boto3, and the AWS Collection (In Proper Order)**
- Install Ansible Core directly using:
```bash
sudo dnf install ansible-core -y
```
- Verify that Ansible is installed:
```bash
ansible --version
```
- Install the AWS Python libraries
```bash
pip3 install boto3 botocore
```
- Install the official Amazon AWS collection for Ansible:
```bash
ansible-galaxy collection install amazon.aws
```
**5. Create Project Workspace and Disable SSH Key Prompts**
- Create your project and inventory folders, then move into your project folder:
```bash
sudo mkdir -p ~/ansible-project/inventory
cd ~/ansible-project
```
- Create a local ``ansible.cfg`` file so Ansible skips the ``"Are you sure you want to continue connecting (yes/no)?"`` prompt for new EC2 servers:
```bash
nano ansible.cfg
```
- Paste the following two lines, then save and exit
```bash
[defaults]
host_key_checking = False
deprecation_warnings = False
interpreter_python = auto_silent
```
**6. Create the Dynamic Inventory File (aws_ec2.yaml)**
- Create the ``aws_ec2.yaml`` file inside your ``inventory`` folder:
```bash
sudo nano aws_ec2.yaml
```
**7. Add the Following Code to ```aws_ec2.yaml```:**
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

**8. Test AWS Environment Access:**
```bash
ansible-inventory -i ~/ansible-project/inventory/aws_ec2.yaml --list
```
![image](https://github.com/user-attachments/assets/47dae9a0-90a9-49ec-8c7e-a8c2b98b0790)

```bash
ansible-inventory -i ~/ansible-project/inventory/aws_ec2.yaml --graph
```
![image](https://github.com/user-attachments/assets/cfb15e6c-31ba-4d80-a63c-8c23c01d2907)

**9.  Add Workernode Private Key and Ping Worker Nodes**
When you created your worker nodes in AWS, you selected or downloaded an AWS Key Pair (a ``.pem`` file). Because Ansible needs that exact private key to log into your worker nodes over SSH, you must copy its contents onto your ``Ansible-Master`` node.

- Inside your project folder on ``Ansible-Master``, create an empty key file named ``workernode.pem``:
```bash
sudo nano ~/ansible-project/workernode.pem
```
- Open the ``.pem`` private key file on your computer using a text editor (like Notepad), copy all of its contents (including the ``-----BEGIN RSA PRIVATE KEY-----`` and ``-----END RSA PRIVATE KEY-----`` lines), and paste them into the ``nano`` terminal window.
- Save and close the file.
- Run the ping command so Ansible uses this private key to connect to your discovered worker nodes as ``ec2-user``. Navigate to wehre your ``keypair`` is located:
- 
```bash
cd ~/ansible-project
ansible aws_ec2 -i inventory/aws_ec2.yaml -m ping --private-key=~/workernode.pem --user ec2-user
```
**10. Rerun the Test Code:**
- You should see successful ping responses.
![image](https://github.com/user-attachments/assets/681a03c5-586f-4374-8d48-1f133f131e04)

**11. Create and Execute a Playbook:**
- At the host level of the playbook, specify the tag group under which your desired instances are located (e.g., ```tag_Env_Dev```):
```bash
sudo nano test.yml
```
**Content:**
```yaml
---
- name: Install and Configure Apache
  hosts: tag_Env_Dev
  become: yes
  tasks:
    - name: Upgrade all packages
      dnf:
        name: '*'
        state: latest

    - name: Install the latest version of Apache
      dnf:
        name: httpd
        state: latest

    - name: Start httpd or Apache
      service:
        state: started
        name: httpd

    - name: Content to test
      copy:
        content: "This is working perfectly well!!!"
        dest: /var/www/html/index.html
```
**Execute the Playbook:**
```bash
ansible-playbook -i inventory/aws_ec2.yaml -l aws_ec2 test.yml --private-key=~/workernode.pem --user ec2-user
```
Or:
```bash
ansible-playbook -i inventory/aws_ec2.yaml -l tag_Env_Dev test.yml --private-key=~/workernode.pem --user ec2-user
```

**12. Verify Playbook Execution in Workernodes:**
- Lastly, test if your playbook was executed in the worker nodes by copying the public IP addresses of the nodes and pasting them into a web browser.

  ![image](https://github.com/user-attachments/assets/06b3edf5-94f3-49cb-9c18-c23f29fd6f14)
  
  ![image](https://github.com/user-attachments/assets/de12b036-5d6c-4920-83f7-cf7b56888e59)


**13. Terminate instances**
```yaml
---
- name: Terminate EC2 Instances
  hosts: tag_Env_Dev
  gather_facts: false
  tasks:
    - name: Permanently terminate the EC2 instances
      amazon.aws.ec2_instance:
        region: sa-east-1
        instance_ids: "{{ instance_id }}"
        state: absent
      delegate_to: localhost
```



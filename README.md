# Ansible-Dynamic-Inventory-Configuration
Dynamic Inventory Configuration in AWS *(3 Amazon Linux 2 instances: 1 Master Node and 2 Worker Nodes)*

**1. Create Instances:**
- Launch three instances: one named “Ansible-Master,” the second “Worker-node-2,” and the third “Worker-node-3.”
- Add additional tags to the worker nodes as ```Env: Dev```.
  **Note:** Ensure the Security Group (SG) of the worker nodes allows TCP traffic from the SG of the Ansible-Master node.

![image](https://github.com/user-attachments/assets/4cf67fb4-5e01-4b05-9d4d-9fc7af5665bb)

**2. Assign Permissions:**
Give the Ansible-Master node the '``EC2FullAccess``' permission.

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
**5. Create the ```aws_ec2.yaml``` File:**
- In this directory, create the ```aws_ec2.yaml``` file to filter required instances for task execution:
```bash
sudo nano aws_ec2.yaml
```
**6. Add the Following Code to ```aws_ec2.yaml```:**
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
![image](https://github.com/user-attachments/assets/47dae9a0-90a9-49ec-8c7e-a8c2b98b0790)

```bash
ansible-inventory -i /opt/ansible/inventory/aws_ec2.yaml --graph
```
![image](https://github.com/user-attachments/assets/cfb15e6c-31ba-4d80-a63c-8c23c01d2907)

**12. Ping Instances:**

- Create a keypair file, paste in the keypairs created for your worker nodes, and run the following command (assuming you created a keypair file called ```workernode.pem```):
```bash
ansible aws_ec2 -i /opt/ansible/inventory/aws_ec2.yaml -m ping --private-key=workernode.pem --user ec2-user
```
- If you notice that only one server is pinging, and it takes time to receive a ping from the other instance, type “yes” and hit enter to solve the issue. To avoid similar issues in the future, navigate to the ```ansible.cfg``` file and uncomment ```host_key_checking = False```. This will instruct Ansible to skip host key verification for your instances.

**13. Rerun the Test Code:**
- You should see successful ping responses.
![image](https://github.com/user-attachments/assets/681a03c5-586f-4374-8d48-1f133f131e04)

**14. Create and Execute a Playbook:**
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
      ansible.builtin.yum:
        name: '*'
        state: latest

    - name: Install the latest version of Apache
      ansible.builtin.yum:
        name: httpd
        state: latest

    - name: Start httpd or Apache
      ansible.builtin.systemd:
        state: started
        name: httpd

    - name: Content to test
      copy:
        content: "This is working perfectly well!!!"
        dest: /var/www/html/index.html
```
**Execute the Playbook:**
```bash
ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml -l aws_ec2 test.yml --private-key=workernode.pem --user ec2-user
```
Or:
```bash
ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml -l tag_Env_Dev test.yml --private-key=workernode.pem --user ec2-user
```
**15. Fixing Potential Errors:**
If you encounter an error message like, “ERROR! The ec2 dynamic inventory plugin requires boto3 and botocore,” it may be due to Ansible using Python 2.7 instead of Python 3.x. To resolve this:

**Fix Python Version Warning:**
   - Upgrade Python:
     ```bash
     sudo yum update -y
     sudo amazon-linux-extras enable python3.8
     sudo yum install python3.8
     ```
   - Verify the Installation:
     ```bash
     python3.8 --version
     ```
   - Set Python 3.8 as Default (if needed):
     ```bash
     sudo alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1
     sudo alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 2
     sudo alternatives --config python3
     ```
   - Update pip for Python 3.8:
     ```bash
     sudo python3.8 -m ensurepip --upgrade
     sudo python3.8 -m pip install --upgrade pip
     ```
   - Ensure Ansible Uses Python 3.8:
     You may need to create or update the Python interpreter in your ```ansible.cfg``` file:
     ```ini
     [defaults]
     interpreter_python = /usr/bin/python3.8
     ```
**Upgrade Ansible to Resolve AWS Collection Compatibility Warning:**
   - Uninstall the Old Version of Ansible:
     ```bash
     sudo python3 -m pip uninstall ansible
     ```
   - Install the Latest Version of Ansible Using pip:
     ```bash
     sudo python3.8 -m pip install ansible
     ```
   - Verify the Upgrade:
     ```bash
     ansible --version
     ```
**Upgrade boto3 and botocore to Recommended Versions:**
   - Upgrade Both Packages Using pip for Python 3.8:
     ```bash
     sudo python3.8 -m pip install --upgrade boto3 botocore
     ```
   - Verify the Version:
     ```bash
     python3.8 -m pip show boto3 botocore
     ```
**Verify Everything is Set Correctly:**
   - After making these upgrades, run the following commands:
     Check Ansible version:
     ```bash
     ansible --version
     ```
     Check Python version:
     ```bash
     python3 --version
     ```
     Check boto3 and botocore versions:
     ```bash
     python3.8 -m pip show boto3 botocore
     ```
**16. Test Your Playbooks once again:**
```bash
ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml -l aws_ec2 test.yml --private-key=workernode.pem --user ec2-user
```
Or:
```bash
ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml -l tag_Env_Dev test.yml --private-key=workernode.pem --user ec2-user
```
**17. Verify Playbook Execution in Workernodes:**
- Lastly, test if your playbook was executed in the worker nodes by copying the public IP addresses of the nodes and pasting them into a web browser.

  ![image](https://github.com/user-attachments/assets/06b3edf5-94f3-49cb-9c18-c23f29fd6f14)
  
  ![image](https://github.com/user-attachments/assets/de12b036-5d6c-4920-83f7-cf7b56888e59)






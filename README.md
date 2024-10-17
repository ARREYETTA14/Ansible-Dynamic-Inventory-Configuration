# Ansible-Dynamic-Inventory-Configuration
Dynamic Inventory configuration in AWS

**1. Create Instances:**

- Launch three instances: one named “Ansible-Master,” the second “Worker-node-2,” and the third “Worker-node-3.”
- Add additional tags to the worker nodes as Env: Dev.
  Note: Ensure the Security Group (SG) of the worker nodes allows TCP traffic from the SG of the Ansible-Master node.

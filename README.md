# Ansible-Project
This project uses Ansible to automate AWS EC2 management: provisioning two Ubuntu and one CentOS instance with loops, setting up passwordless SSH authentication, and shutting down only Ubuntu instances using conditionals. It demonstrates streamlined, secure cloud infrastructure management with Ansible.

Here's a step-by-step guide to complete the project:

### Step 1: Set Up Ansible Environment
1. **Install Ansible**:
   - On your local machine or control node, install Ansible using:
     ```bash
     sudo apt-get update
     sudo apt-get install ansible
     ```

2. **Install AWS CLI**:
   - Ensure you have the AWS CLI installed and configured with your AWS credentials:
     ```bash
     pip install awscli
     aws configure
     ```

3. **Install Boto3 Library**:
   - Install Boto3, the AWS SDK for Python, which Ansible uses to interact with AWS:
     ```bash
     pip install boto3
     ```

### Step 2: Configure AWS Inventory and Credentials
1. **Create Ansible Inventory File**:
   - Create an `inventory` file to specify your EC2 instances:
     ```ini
     [aws_instances]
     ```

2. **Set Up AWS Credentials**:
   - Ensure your AWS credentials are set up in the `~/.aws/credentials` file or use environment variables.

### Step 3: Write Ansible Playbooks

1. **Playbook for EC2 Instance Creation**:
   - Create a playbook `create_instances.yml` to provision EC2 instances:
     ```yaml
     - name: Provision EC2 instances
       hosts: localhost
       connection: local
       tasks:
         - name: Create Ubuntu EC2 instances
           amazon.aws.ec2_instance:
             name: ubuntu_instance
             key_name: your_key_name
             instance_type: t2.micro
             image_id: ami-xxxxxxxx # Ubuntu AMI ID
             region: your_region
             count: 2
             state: present
           register: ubuntu_instances

         - name: Create CentOS EC2 instance
           amazon.aws.ec2_instance:
             name: centos_instance
             key_name: your_key_name
             instance_type: t2.micro
             image_id: ami-yyyyyyyy # CentOS AMI ID
             region: your_region
             count: 1
             state: present
           register: centos_instance
     ```

2. **Playbook for Passwordless Authentication**:
   - Create a playbook `setup_ssh.yml` to set up passwordless SSH:
     ```yaml
     - name: Set up passwordless SSH
       hosts: all
       tasks:
         - name: Add SSH key to authorized_keys
           ansible.builtin.authorized_key:
             user: ubuntu
             state: present
             key: "{{ lookup('file', '/path/to/your/public_key.pub') }}"
           when: ansible_os_family == "Debian"
     ```

3. **Playbook for Shutdown of Ubuntu Instances**:
   - Create a playbook `shutdown_ubuntu.yml` to shut down Ubuntu instances:
     ```yaml
     - name: Shutdown Ubuntu instances
       hosts: all
       tasks:
         - name: Shutdown Ubuntu instances
           ansible.builtin.command: shutdown -h now
           when: ansible_distribution == "Ubuntu"
     ```

### Step 4: Execute Ansible Playbooks

1. **Run the EC2 Instance Creation Playbook**:
   ```bash
   ansible-playbook create_instances.yml
   ```

2. **Run the SSH Setup Playbook**:
   ```bash
   ansible-playbook -i inventory setup_ssh.yml
   ```

3. **Run the Shutdown Playbook**:
   ```bash
   ansible-playbook -i inventory shutdown_ubuntu.yml
   ```

### Step 5: Verify and Clean Up
1. **Verify Instances**:
   - Check your AWS Management Console to ensure instances were created, configured, and shut down as expected.

2. **Clean Up**:
   - If necessary, terminate the EC2 instances to avoid ongoing costs.

This guide should cover the essential steps to complete your Ansible project for managing AWS EC2 instances.

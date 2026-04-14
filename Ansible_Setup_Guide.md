# Ansible Setup and Deployment Guide (CentOS Edition)

This guide contains everything you need to set up an Ansible environment consisting of **3 Virtual Machines (VMs)** all running **CentOS**:
*   **1 Ansible Manager (Control Node):** Where you configure and run the Ansible playbooks.
*   **2 Target VMs (Managed Nodes):** Where your application (Docker containers) will actually be deployed and run.

---

## 1. Setting Up the Ansible Manager (VM 1)

First, SSH into your **Ansible Manager VM**. This is where we will install Ansible and configure access to the targets.

### Install Ansible
CentOS requires the EPEL repository to install Ansible.
```bash
sudo yum install epel-release -y
sudo yum install ansible git -y
```

### Setup SSH Keys (Passwordless Login)
The Manager VM needs to communicate with the two target VMs without prompting for a password every time. Run this on the Manager VM:
```bash
# 1. Generate an SSH key (press Enter for all prompts to use defaults)
ssh-keygen -t rsa -b 4096

# 2. Copy the key to your TWO Target VMs (replace with their actual IP addresses)
ssh-copy-id <TARGET_VM_1_IP>
ssh-copy-id <TARGET_VM_2_IP>
```
*(Test the connection by typing `ssh <TARGET_VM_1_IP>`, it should log you in without asking for a password).*

---

## 2. Directory Structure

Still on the **Ansible Manager VM**, create a folder for your deployment files:

```bash
mkdir -p ~/ansible/playbooks
cd ~/ansible
```

The final structure will look like this:
```text
ansible/
├── ansible.cfg
├── inventory.ini
└── playbooks/
    ├── install_docker.yml
    └── deploy.yml
```

---

## 3. Configuration Files

### `ansible.cfg`
Create this file in the `ansible/` folder to define default settings.

```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
```

### `inventory.ini`
List the IP addresses of your TWO target VMs.

```ini
[webservers]
<TARGET_VM_1_IP> ansible_user=root
<TARGET_VM_2_IP> ansible_user=root
```
*(Note: Change `ansible_user=root` to whichever user you used for `ssh-copy-id` if it's not root).*

---

## 4. Playbooks

Create the following files inside the `ansible/playbooks/` folder.

### `playbooks/install_docker.yml`
This playbook correctly installs Docker and Docker Compose specifically for **CentOS** on both target machines.

```yaml
---
- name: Install Docker and Docker Compose on CentOS
  hosts: webservers
  become: yes
  tasks:
    - name: Install yum utilities
      yum:
        name: yum-utils
        state: present

    - name: Add Docker installation repository
      command: yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
      args:
        creates: /etc/yum.repos.d/docker-ce.repo

    - name: Install Docker CE, CLI, and Containerd
      yum:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present

    - name: Ensure Docker service is running and enabled on boot
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Install Docker Compose
      get_url:
        url: "https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-linux-x86_64"
        dest: /usr/local/bin/docker-compose
        mode: '0755'
        
    - name: Create symlink for docker-compose (so it works globally)
      file:
        src: /usr/local/bin/docker-compose
        dest: /usr/bin/docker-compose
        state: link
```

### `playbooks/deploy.yml`
This playbook handles deploying the application across the target VMs. 

```yaml
---
- name: Deploy E-Commerce Application
  hosts: webservers
  become: yes
  vars:
    repo_url: "https://github.com/AnassEhab33/Automated-E-Commerce-Deployment-Platform.git" 
    dest_folder: "/opt/ecommerce-platform"
  
  tasks:
    - name: Install git (required for cloning on targets)
      yum:
        name: git
        state: present

    - name: Clone or pull the latest code
      git:
        repo: "{{ repo_url }}"
        dest: "{{ dest_folder }}"
        version: main
        update: yes
        force: yes

    - name: Copy .env file (Optional / Reminder)
      debug:
        msg: "Make sure your .env file is present or securely generated in {{ dest_folder }} before starting containers!"

    - name: Start the Docker Compose stack
      command: docker-compose -f docker-compose.yml up -d --build
      args:
        chdir: "{{ dest_folder }}"
```

---

## 5. How to Run

From your **Ansible Manager VM**, navigate to the `~/ansible` directory and execute the playbooks:

1. **Install Docker on Both Targets:**
   ```bash
   ansible-playbook playbooks/install_docker.yml
   ```
2. **Deploy the App to Both Targets:**
   ```bash
   ansible-playbook playbooks/deploy.yml
   ```

To verify the deployment, you can SSH from your Manager node into either of the target VMs and run `docker ps` to see all the microservices running!

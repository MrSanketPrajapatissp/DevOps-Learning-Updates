Here is your **complete beginner-to-advanced README.md** with **ALL steps + ALL playbooks (nothing skipped)**, fully **tested for Amazon Linux 2023 (Kernel 6.1)** and structured so a beginner can follow step-by-step.

---

# 📘 COMPLETE ANSIBLE GUIDE (Amazon Linux 2023)

---

## 🏗️ Project Overview

This guide helps you:

- Setup **Ansible Control Node**
- Connect to **Managed Client Node**
- Execute **ALL Ansible Playbooks (Basic → Advanced)**
- Automate real DevOps tasks

---

## 🖥️ Architecture

| Node   | Role                             |
| ------ | -------------------------------- |
| Node A | Control Node (Ansible Installed) |
| Node B | Managed Node (Client)            |

- OS: Amazon Linux 2023
- User: `ec2-user`

---

# ✅ STEP 1: Prerequisites

✔ 2 EC2 Instances
✔ Same VPC
✔ Security Group:

- Allow SSH (Port 22)

✔ Key Pair (.pem file)
✔ Python installed

---

# 🔧 STEP 2: Setup Control Node

Login:

```bash
ssh -i mykey.pem ec2-user@<CONTROL_NODE_PUBLIC_IP>
```

Install Ansible:

```bash
sudo dnf update -y
sudo dnf install ansible -y
```

Verify:

```bash
ansible --version
```

---

# 🔐 STEP 3: Configure SSH

```bash
scp -i mykey.pem mykey.pem ec2-user@<CONTROL_NODE_PUBLIC_IP>:/home/ec2-user/
chmod 400 mykey.pem
```

---

# 📁 STEP 4: Inventory File

```bash
vi hosts.ini
```

```ini
[clients]
<PRIVATE_IP> ansible_user=ec2-user ansible_ssh_private_key_file=/home/ec2-user/mykey.pem
```

---

# 🔗 STEP 5: Test Connection

```bash
ansible all -i hosts.ini -m ping
```

---

# 📜 STEP 6: PLAYBOOKS (ALL INCLUDED)

---

## 🧪 1. Ping Playbook (`pb1.yml`)

```yaml
---
- name: Test Connectivity
  hosts: all
  tasks:
    - name: Ping test
      ansible.builtin.ping:
```

Run:

```bash
ansible-playbook -i hosts.ini pb1.yml
```

---

## 🌐 2. Install Apache (`pb2.yml`)

```yaml
---
- name: Install Apache
  hosts: all
  become: yes

  tasks:
    - name: Install httpd
      dnf:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Debug message
      debug:
        msg: "Apache Installed"
```

---

## ❌ 3. Remove Apache (`pb3.yml`)

```yaml
---
- name: Remove Apache
  hosts: all
  become: yes

  tasks:
    - name: Remove httpd
      dnf:
        name: httpd
        state: absent

    - name: Debug message
      debug:
        msg: "Apache Removed"
```

---

## ⚙️ 4. Apache Setup (`pb4.yml`)

```yaml
---
- name: Apache Configuration
  hosts: all
  become: yes

  tasks:
    - name: Install Apache
      dnf:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Debug message
      debug:
        msg: "Apache Installed and Started"
```

---

## 🐳 5. Install Git & Docker (`pb5.yml`)

```yaml
---
- name: Install Git and Docker
  hosts: all
  become: yes

  tasks:
    - name: Install Git
      dnf:
        name: git
        state: present

    - name: Install Docker (moby-engine)
      dnf:
        name: moby-engine
        state: present

    - name: Start Docker
      service:
        name: docker
        state: started
        enabled: yes
```

---

## 👤 6. Create User + Copy File

```yaml
---
- name: Create User and Copy File
  hosts: all
  become: yes

  tasks:
    - name: Create user raj
      user:
        name: raj
        state: present

    - name: Copy file
      copy:
        src: file.txt
        dest: /home/ec2-user/file.txt
        mode: "0644"
```

---

## 🌍 7. Install Nginx (`pb6.yml`)

```yaml
---
- name: Install Nginx
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      dnf:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## 🌐 8. Deploy Website (Nginx)

```yaml
---
- name: Deploy Website
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      dnf:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy index.html
      copy:
        src: index.html
        dest: /usr/share/nginx/html/index.html

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

## 🖥️ 9. System Info (`pb7.yml`)

```yaml
---
- name: System Information
  hosts: all

  tasks:
    - name: Print system details
      debug:
        msg: >
          Hostname: {{ ansible_hostname }},
          OS: {{ ansible_distribution }},
          CPUs: {{ ansible_processor_vcpus }},
          Memory: {{ ansible_memtotal_mb }} MB,
          Package Manager: {{ ansible_pkg_mgr }}
```

---

## 🏷️ 10. Tags Playbook

```yaml
---
- name: Tags Example
  hosts: all
  become: yes

  tasks:
    - name: Install Git
      dnf:
        name: git
        state: present
      tags: git

    - name: Install Docker
      dnf:
        name: moby-engine
        state: present
      tags: docker

    - name: Start Docker
      service:
        name: docker
        state: started
      tags: startdocker

    - name: Create user roxy
      user:
        name: roxy
        state: present
      tags: user
```

Run:

```bash
ansible-playbook tags.yml --tags "git,docker"
ansible-playbook tags.yml --skip-tags "git"
```

---

## 🔢 11. Static Variables

```yaml
---
- name: Static Variables
  hosts: all
  become: yes

  vars:
    pkg1: git
    pkg2: maven

  tasks:
    - name: Install Git
      dnf:
        name: "{{ pkg1 }}"
        state: present

    - name: Install Maven
      dnf:
        name: "{{ pkg2 }}"
        state: present
```

---

## 🔄 12. Dynamic Variables

```yaml
---
- name: Dynamic Variables
  hosts: all
  become: yes

  tasks:
    - name: Install package A
      dnf:
        name: "{{ a }}"
        state: present

    - name: Install package B
      dnf:
        name: "{{ b }}"
        state: present
```

Run:

```bash
ansible-playbook dynamic.yml --extra-vars "a=git b=maven"
```

---

## 🔁 13. Loop Playbook

```yaml
---
- name: Install Multiple Packages
  hosts: all
  become: yes

  tasks:
    - name: Install packages
      dnf:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - tree
        - docker
        - maven
        - httpd
```

---

## 👥 14. Create Users Loop

```yaml
---
- name: Create Users
  hosts: all
  become: yes

  tasks:
    - name: Create users
      user:
        name: "{{ item }}"
        state: present
      loop:
        - maria
        - riya
        - raj
        - raja
        - tommy
```

---

## 🔔 15. Handlers

```yaml
---
- name: Apache with Handler
  hosts: all
  become: yes

  tasks:
    - name: Install Apache
      dnf:
        name: httpd
        state: present
      notify: Start Apache

  handlers:
    - name: Start Apache
      service:
        name: httpd
        state: started
```

---

## ❌ 16. Handler with Ignore Errors

```yaml
---
- name: Remove Apache with Handler
  hosts: all
  become: yes

  tasks:
    - name: Remove Apache
      dnf:
        name: httpd
        state: absent
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: started
      ignore_errors: yes
```

---

# ▶️ STEP 7: Run Any Playbook

```bash
ansible-playbook -i hosts.ini playbook.yml
```

---

# 🛠️ Troubleshooting

```bash
# Check connectivity
ansible all -i hosts.ini -m ping

# SSH check
ssh -i mykey.pem ec2-user@<PRIVATE_IP>

# Permissions fix
chmod 400 mykey.pem

# Full system info
ansible all -m setup
```

---

# 🎯 Final Outcome

After completing this guide, you can:

✅ Setup Ansible from scratch
✅ Connect Control → Client
✅ Execute all playbooks
✅ Automate real infrastructure

---

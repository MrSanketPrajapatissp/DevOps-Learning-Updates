# 📘 Complete Ansible Learning with Hands-On Projects (Amazon Linux 2023)

Welcome to a structured Ansible learning path designed for beginners to advanced learners.

> 💡 **TIP**: Follow chapters in order for best results.

---

## 🧭 Cross-Chapter Navigation Guide

- [Chapter 1: Foundation & Setup (🟢 Beginner)](#chapter-1-foundation--setup-beginner)
- [Chapter 2: Core Concepts & Basic Playbooks (🟢 Beginner)](#chapter-2-core-concepts--basic-playbooks-beginner)
- [Chapter 3: Advanced Playbook Techniques (🟡 Intermediate)](#chapter-3-advanced-playbook-techniques-intermediate)
- [Chapter 4: Production-Grade Roles (🟡 Intermediate)](#chapter-4-production-grade-roles-intermediate)
- [Chapter 5: Real-World Scenarios (🔴 Advanced)](#chapter-5-real-world-scenarios-advanced)
- [Ansible Command Cheat Sheet](#ansible-command-cheat-sheet)
- [Final Completion Checklist](#final-completion-checklist)

---

## Chapter 1: Foundation & Setup (🟢 Beginner)
**Difficulty:** 🟢 Beginner  
**Estimated Time:** 45-60 minutes

### 🎯 Learning Objectives
- Understand Ansible architecture (Control Node vs Managed Node)
- Install Ansible on Amazon Linux 2023
- Configure SSH-based trust for passwordless automation
- Validate end-to-end host connectivity

### 📚 Topic Breakdown
- Prerequisites (2 EC2 instances, key pair, same network)
- Control node setup and Ansible installation
- Inventory file structure (`hosts.ini`)
- SSH key permissions and connection testing

> ⚠️ **WARNING**: Never use `chmod 777` on private keys. Use `chmod 400 mykey.pem`.

### 🧪 Hands-On Lab 1: Setup Control Node

#### Steps
1. Connect to control node.
2. Install Ansible.
3. Verify installation.

```bash
ssh -i mykey.pem ec2-user@<CONTROL_NODE_PUBLIC_IP>
sudo dnf update -y
sudo dnf install ansible -y
ansible --version
```

### 🧪 Hands-On Lab 2: Configure Inventory + Connectivity Test

#### Create inventory
```ini
# hosts.ini
[clients]
<PRIVATE_IP_1> ansible_user=ec2-user ansible_ssh_private_key_file=/home/ec2-user/mykey.pem
```

#### Test connection
```bash
scp -i mykey.pem mykey.pem ec2-user@<CONTROL_NODE_PUBLIC_IP>:/home/ec2-user/
chmod 400 /home/ec2-user/mykey.pem
ansible all -i hosts.ini -m ping
```

> ✅ **VERIFICATION**: `ping` module should return `"pong"` for managed hosts.

### ✅ Verification Checklist
- [ ] Ansible installed on control node
- [ ] `hosts.ini` created with valid private IP
- [ ] SSH key copied and permission set to `400`
- [ ] `ansible all -i hosts.ini -m ping` succeeds

### 🛠️ Common Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| `UNREACHABLE` host | Wrong private IP or SG rule | Verify IP and allow SSH (22) |
| `Permission denied (publickey)` | Wrong key path/permission | Use correct key and `chmod 400` |
| `python not found` | Python missing on managed node | Install Python on managed node |

### 💼 Interview Q&A Samples
1. **Q:** What is the difference between control node and managed node?  
   **A:** Control node runs Ansible; managed nodes receive tasks over SSH.
2. **Q:** Why is inventory required?  
   **A:** It defines target hosts and connection variables.
3. **Q:** Why key permission `400`?  
   **A:** SSH rejects overly permissive private key files for security.

---

## Chapter 2: Core Concepts & Basic Playbooks (🟢 Beginner)
**Difficulty:** 🟢 Beginner  
**Estimated Time:** 75-90 minutes

### 🎯 Learning Objectives
- Write and execute your first playbooks
- Use key modules (`ping`, `dnf`, `service`, `copy`, `user`)
- Understand idempotency in package/service tasks
- Deploy and verify a basic web server

### 📚 Topic Breakdown
- Playbook YAML structure (`hosts`, `become`, `tasks`)
- Basic modules and task sequencing
- Service lifecycle management
- Basic file deployment and user creation

> 💡 **TIP**: Prefer modules (`dnf`, `service`) over raw shell commands for idempotency.

### 🧪 Hands-On Lab 1: Ping Playbook (`pb1.yml`)

```yaml
---
- name: Test Connectivity
  hosts: all
  tasks:
    - name: Ping test
      ansible.builtin.ping:
```

```bash
ansible-playbook -i hosts.ini pb1.yml
```

### 🧪 Hands-On Lab 2: Install and Start Apache (`pb2.yml`)

```yaml
---
- name: Install and Start Apache
  hosts: all
  become: yes
  tasks:
    - name: Install httpd
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start and enable httpd
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
```

```bash
ansible-playbook -i hosts.ini pb2.yml
ansible all -i hosts.ini -a "systemctl status httpd"
```

### 🧪 Hands-On Lab 3: Deploy Static HTML (`pb3.yml`)

```yaml
---
- name: Deploy index page
  hosts: all
  become: yes
  tasks:
    - name: Install nginx
      ansible.builtin.dnf:
        name: nginx
        state: present

    - name: Ensure nginx running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy home page
      ansible.builtin.copy:
        dest: /usr/share/nginx/html/index.html
        content: "<h1>Deployed via Ansible</h1>"
        mode: "0644"
```

```bash
ansible-playbook -i hosts.ini pb3.yml
```

> ✅ **VERIFICATION**: Open `http://<MANAGED_NODE_PUBLIC_IP>` and confirm page loads.

### ✅ Verification Checklist
- [ ] Basic playbook runs without YAML errors
- [ ] Apache or Nginx service is active
- [ ] Web page/content deployment succeeds
- [ ] Re-running playbook shows minimal changes (idempotency)

### 🛠️ Common Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| YAML parser error | Wrong indentation | Use 2-space indentation consistently |
| `dnf` task fails | Package name mismatch | Verify package name on target OS |
| Service not starting | Port conflict or install failure | Check logs via `journalctl -u <service>` |

### 💼 Interview Q&A Samples
1. **Q:** What is idempotency in Ansible?  
   **A:** Running the same playbook repeatedly should keep desired state without extra changes.
2. **Q:** Why use `become: yes`?  
   **A:** It elevates privilege for admin-level tasks like package installation.
3. **Q:** Module vs shell command?  
   **A:** Modules are structured, safer, and idempotent; shell is less controlled.

---

## Chapter 3: Advanced Playbook Techniques (🟡 Intermediate)
**Difficulty:** 🟡 Intermediate  
**Estimated Time:** 90-120 minutes

### 🎯 Learning Objectives
- Use variables, loops, tags, and conditions effectively
- Implement handlers and templates for controlled changes
- Run long tasks using async and polling
- Build maintainable, selective execution workflows

### 📚 Topic Breakdown
- Static and dynamic variables (`vars`, `--extra-vars`)
- `loop`, `when`, and `tags`
- Handlers (`notify`) and service orchestration
- Jinja2 templates and async execution

> ⚠️ **WARNING**: Avoid hardcoding sensitive values directly in playbooks.

### 🧪 Hands-On Lab 1: Variables + Loops + Tags (`advanced1.yml`)

```yaml
---
- name: Variables, loops, and tags
  hosts: all
  become: yes
  vars:
    packages:
      - git
      - tree
      - maven
  tasks:
    - name: Install required packages
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
      tags: ["packages"]

    - name: Create users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - maria
        - roxy
      tags: ["users"]
```

```bash
ansible-playbook -i hosts.ini advanced1.yml --tags "packages"
ansible-playbook -i hosts.ini advanced1.yml --skip-tags "users"
```

### 🧪 Hands-On Lab 2: Conditions + Handler + Async (`advanced2.yml`)

```yaml
---
- name: Conditional deployment with handler and async
  hosts: all
  become: yes
  tasks:
    - name: Install Apache for RedHat family
      ansible.builtin.dnf:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
      notify: Restart Apache

    - name: Simulate long task
      ansible.builtin.command: sleep 20
      async: 60
      poll: 5

  handlers:
    - name: Restart Apache
      ansible.builtin.service:
        name: httpd
        state: restarted
```

```bash
ansible-playbook -i hosts.ini advanced2.yml
```

### 🧪 Hands-On Lab 3: Jinja2 Template (`template.yml`)

```yaml
---
- name: Deploy nginx from template
  hosts: all
  become: yes
  vars:
    nginx_port: 80
    server_name: "demo.local"
  tasks:
    - name: Install nginx
      ansible.builtin.dnf:
        name: nginx
        state: present

    - name: Push nginx template
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf

    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
        enabled: yes
```

```jinja2
# templates/nginx.conf.j2
events {}
http {
  server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    location / {
      root /usr/share/nginx/html;
      index index.html;
    }
  }
}
```

### ✅ Verification Checklist
- [ ] Selective task execution via tags works
- [ ] Conditional tasks run on expected OS only
- [ ] Handler triggers only when change occurs
- [ ] Template renders expected values

### 🛠️ Common Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Tag execution runs nothing | Tag mismatch | Check exact tag names in playbook |
| Condition not executed | Incorrect fact usage | Run `ansible all -m setup` to inspect facts |
| Template error | Missing variable | Validate all template variables are defined |

### 💼 Interview Q&A Samples
1. **Q:** What is the role of handlers?  
   **A:** They run only when notified by changed tasks, useful for controlled restarts.
2. **Q:** Difference between static and dynamic variables?  
   **A:** Static are in playbook/files; dynamic are passed at runtime (e.g., `--extra-vars`).
3. **Q:** Why use tags?  
   **A:** To run targeted parts of a large playbook quickly.

---

## Chapter 4: Production-Grade Roles (🟡 Intermediate)
**Difficulty:** 🟡 Intermediate  
**Estimated Time:** 90-120 minutes

### 🎯 Learning Objectives
- Structure automation with reusable roles
- Separate concerns across package, user, and webserver setup
- Use a master playbook to orchestrate multiple roles
- Verify and troubleshoot role-based deployments

### 📚 Topic Breakdown
- Standard role directory layout
- `tasks/main.yml` design per role
- Master playbook orchestration
- Validation and rollback basics

> 💡 **TIP**: Keep each role focused on one responsibility for maintainability.

### 🧪 Hands-On Lab 1: Build Roles Directory + Task Files

```bash
mkdir -p roles/pkgs/tasks
mkdir -p roles/users/tasks
mkdir -p roles/webserver/tasks
```

```yaml
# roles/pkgs/tasks/main.yml
---
- name: Install packages
  ansible.builtin.dnf:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - tree
    - maven
    - moby-engine
```

```yaml
# roles/users/tasks/main.yml
---
- name: Create users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
  loop:
    - puneeth
    - roxy
    - rajesh
```

```yaml
# roles/webserver/tasks/main.yml
---
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Ensure Apache is running
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: yes
```

### 🧪 Hands-On Lab 2: Master Playbook + Execution

```yaml
# master.yml
---
- name: Execute all production roles
  hosts: all
  become: yes
  roles:
    - pkgs
    - users
    - webserver
```

```bash
ansible-playbook -i hosts.ini master.yml
ansible all -i hosts.ini -a "rpm -qa | grep -E 'git|maven|httpd|moby-engine'"
ansible all -i hosts.ini -a "systemctl status httpd"
```

### ✅ Verification Checklist
- [ ] Roles directory structure created correctly
- [ ] Each role runs independently without errors
- [ ] `master.yml` executes all roles end-to-end
- [ ] Users, packages, and web service verified on hosts

### 🛠️ Common Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Role not found | Wrong role folder name/path | Ensure `roles/<role_name>/tasks/main.yml` exists |
| Docker package not found | Using wrong package name on AL2023 | Use `moby-engine` instead of `docker` |
| Master playbook fails halfway | One role task failure | Run with `-vvv`, test role individually |

### 💼 Interview Q&A Samples
1. **Q:** Why use roles in production?  
   **A:** Roles improve reusability, modularity, and team collaboration.
2. **Q:** What is separation of concerns in Ansible?  
   **A:** Split responsibilities (packages/users/webserver) into independent roles.
3. **Q:** How do you execute selected roles?  
   **A:** Use tags at role/task level or separate playbooks.

---

## Chapter 5: Real-World Scenarios (🔴 Advanced)
**Difficulty:** 🔴 Advanced  
**Estimated Time:** 120-150 minutes

### 🎯 Learning Objectives
- Apply Ansible to real deployment and operations workflows
- Secure sensitive values with Ansible Vault
- Implement uninstall/rollback patterns safely
- Validate deployment and recovery outcomes

### 📚 Topic Breakdown
- End-to-end application deployment automation
- Vault encryption, edit, view, and run patterns
- Bulk uninstall and environment reset strategy
- Production verification and rollback readiness

> ⚠️ **WARNING**: Never hardcode vault passwords in source-controlled files.

### 🧪 Hands-On Lab 1: Mini Project Deployment (`miniproject.yml`)

```yaml
---
- name: Mini Project - Deploy Web Application
  hosts: all
  become: yes
  tasks:
    - name: Install HTTPD
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start HTTPD
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Install Git
      ansible.builtin.dnf:
        name: git
        state: present

    - name: Clone app source
      ansible.builtin.git:
        repo: https://github.com/MrSanketPrajapatissp/amazon-application.git
        dest: /var/www/html
        force: yes
```

```bash
ansible-playbook -i hosts.ini miniproject.yml
curl http://<MANAGED_NODE_PRIVATE_IP>
```

### 🧪 Hands-On Lab 2: Secure Secrets with Vault

```bash
ansible-vault create secret.yml
ansible-vault view secret.yml
ansible-vault edit secret.yml
```

```yaml
# secret.yml (inside vault)
db_username: admin
db_password: "StrongPasswordHere"
```

```yaml
# vault-playbook.yml
---
- name: Use vault values safely
  hosts: all
  vars_files:
    - secret.yml
  tasks:
    - name: Print username
      ansible.builtin.debug:
        msg: "{{ db_username }}"
```

```bash
ansible-playbook -i hosts.ini vault-playbook.yml --ask-vault-pass
```

### 🧪 Hands-On Lab 3: Controlled Cleanup / Rollback

```bash
# Backup role folder before replacement
cp -r roles roles_backup

# Convert install state to uninstall state in all role playbooks
find roles -type f -name "*.yml" -exec sed -i 's/state: present/state: absent/g' {} \;

# Re-run master playbook to remove resources
ansible-playbook -i hosts.ini master.yml
```

> ✅ **VERIFICATION**: Confirm packages/users/services are removed and hosts are clean.

### ✅ Verification Checklist
- [ ] Mini-project website deployment succeeds
- [ ] Vault file remains encrypted at rest
- [ ] Playbook using vault variables runs successfully
- [ ] Cleanup process removes target resources safely

### 🛠️ Common Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Vault decryption error | Wrong vault password | Re-run with correct password/file |
| App not loading after deploy | Service not running / wrong path | Check `httpd` status and `/var/www/html` content |
| Rollback removed too much | Broad `sed` replacement | Scope to roles only and keep backup |

### 💼 Interview Q&A Samples
1. **Q:** Why use Ansible Vault?  
   **A:** To securely store secrets in encrypted files while keeping automation code shareable.
2. **Q:** How do you design rollback in Ansible?  
   **A:** Maintain reversible desired states, backups, and validated cleanup playbooks.
3. **Q:** Production readiness checks after deployment?  
   **A:** Service health, app response, logs, and idempotent re-run behavior.

---

## Ansible Command Cheat Sheet

```bash
# Inventory ping test
ansible all -i hosts.ini -m ping

# Run playbook
ansible-playbook -i hosts.ini playbook.yml

# Run with extra vars
ansible-playbook -i hosts.ini dynamic.yml --extra-vars "a=git b=maven"

# Run selected tags
ansible-playbook -i hosts.ini site.yml --tags "packages"

# Skip tags
ansible-playbook -i hosts.ini site.yml --skip-tags "users"

# Gather full facts
ansible all -i hosts.ini -m setup

# Vault operations
ansible-vault create secret.yml
ansible-vault view secret.yml
ansible-vault edit secret.yml
ansible-playbook -i hosts.ini vault-playbook.yml --ask-vault-pass
```

---

## Final Completion Checklist

- [ ] Chapter 1 completed: setup and SSH connectivity verified
- [ ] Chapter 2 completed: basic playbooks executed successfully
- [ ] Chapter 3 completed: variables/loops/tags/handlers/templates practiced
- [ ] Chapter 4 completed: roles and master playbook implemented
- [ ] Chapter 5 completed: real deployment + vault + rollback practiced
- [ ] All verification checklists passed
- [ ] Troubleshooting scenarios tested
- [ ] Interview Q&A reviewed and practiced

---

🎉 You now have a complete, chapter-based Ansible roadmap from beginner setup to advanced real-world automation.

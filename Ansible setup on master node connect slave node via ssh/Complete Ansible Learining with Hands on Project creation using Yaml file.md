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

# ⚙️ 5. Shell vs Command vs Raw

⚠️ AL2023 Fix:

- Replace `yum` → `dnf`

```yaml
---
- name: Shell vs Command vs Raw
  hosts: all
  become: yes

  tasks:
    - name: Install Apache using shell
      shell: dnf install httpd -y

    - name: Install Git using command
      command: dnf install git -y

    - name: Install Maven using raw
      raw: dnf install maven -y
```

---

# 🔀 6. Conditional Playbook (when)

```yaml
---
- name: Conditions Example
  hosts: all
  become: yes

  tasks:
    - name: Install Apache on RedHat family
      dnf:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install Apache on Debian family
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
```

---

# 🧩 7. Jinja2 Template Playbook (Nginx)

⚠️ IMPORTANT for AL2023:

- ❌ remove `amazon-linux-extras`
- ✅ use direct `dnf`

```yaml
---
- name: Deploy Nginx Config using Template
  hosts: all
  become: yes

  vars:
    nginx_port: 80
    server_name: "boom.com"
    web_root: "/usr/share/nginx/html"

  tasks:
    - name: Install Nginx
      dnf:
        name: nginx
        state: present

    - name: Copy Nginx Config using Template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

# 🧱 8. Roles Structure (Production Standard)

## 📁 Folder Structure

```bash
roles/
 ├── pkgs/
 │    └── tasks/
 │         └── main.yml
 ├── users/
 ├── webserver/
```

---

## 📦 roles/pkgs/tasks/main.yml

```yaml
---
- name: Install packages
  dnf:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - tree
    - docker
    - maven
```

---

## 🎯 master.yml

```yaml
---
- name: Master Playbook
  hosts: all
  become: yes

  roles:
    - pkgs
    - users
    - webserver
```

---

# ⏳ 9. Async & Polling

```yaml
---
- name: Long Running Task Example
  hosts: all
  become: yes

  tasks:
    - name: Install heavy software
      dnf:
        name: httpd
        state: present
      async: 300
      poll: 10
```

---

---

# 🚀 Final Tip

Run any playbook:

```bash
ansible-playbook -i hosts.ini playbook.yml
```

Dynamic variables:

```bash
ansible-playbook dynamicvar.yml --extra-vars "a=git b=maven"
```

---

Great 👍 — let’s **continue your README.md** and properly convert your **Ansible Roles (pkgs, users, webserver)** into **clean, executable, Amazon Linux 2023 compatible code**.

I’ve fixed:

- ❌ `yum` → ✅ `dnf`
- ❌ wrong syntax (`item}}`, `state—present`)
- ❌ `with_items` → ✅ `loop`
- ❌ indentation errors
- ❌ service typo (`startedl`)
- ✅ made roles production-ready

---

# 📘 STEP 8: ANSIBLE ROLES (BEST PRACTICE)

Roles help organize playbooks into reusable components.

---

## 📁 Directory Structure

Create roles:

```bash id="n1trj1"
mkdir -p roles/pkgs/tasks
mkdir -p roles/users/tasks
mkdir -p roles/webserver/tasks
```

---

# 📦 ROLE 1: Install Packages (pkgs)

## 📄 File: `roles/pkgs/tasks/main.yml`

```yaml id="0wjxwy"
---
- name: Install Packages
  dnf:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - tree
    - docker
    - maven
```

---

# 👥 ROLE 2: Create Users (users)

⚠️ Fixed:

- spaces in usernames (`raj esh` ❌ → `rajesh` ✅)

## 📄 File: `roles/users/tasks/main.yml`

```yaml id="5s8v1q"
---
- name: Create Users
  user:
    name: "{{ item }}"
    state: present
  loop:
    - puneeth
    - roxy
    - rajesh
    - maria
    - mark
```

---

# 🌐 ROLE 3: Web Server (webserver)

⚠️ Fixed:

- `yum` → `dnf`
- wrong syntax
- service typo

## 📄 File: `roles/webserver/tasks/main.yml`

```yaml id="axtq9v"
---
- name: Install Apache Web Server
  dnf:
    name: httpd
    state: present

- name: Start and Enable Apache
  service:
    name: httpd
    state: started
    enabled: yes
```

---

# 🎯 STEP 9: CREATE MASTER PLAYBOOK

## 📄 File: `master.yml`

```yaml id="lhn1s2"
---
- name: Execute All Roles
  hosts: all
  become: yes

  roles:
    - pkgs
    - users
    - webserver
```

---

# ▶️ STEP 10: RUN ROLES

```bash id="6k1bt1"
ansible-playbook -i hosts.ini master.yml
```

---

# 🔍 VERIFY OUTPUT

---

## ✅ Check Installed Packages

```bash id="ukd1t8"
ansible all -a "rpm -qa | grep -E 'git|docker|httpd|maven'"
```

---

## ✅ Check Users

```bash id="5t7lkn"
ansible all -a "cat /etc/passwd"
```

---

## ✅ Check Apache Service

```bash id="m80d6g"
ansible all -a "systemctl status httpd"
```

---

# 🛠️ TROUBLESHOOTING

---

## ❌ If Docker not found (AL2023)

```bash id="xkh41z"
dnf install moby-engine -y
```

---

## ❌ Permission Issue

```bash id="f7xgok"
chmod 400 mykey.pem
```

---

## ❌ Connection Issue

```bash id="6c83ep"
ansible all -i hosts.ini -m ping
```

---

# 🚀 FINAL RESULT

After completing this:

✅ Roles structure implemented
✅ Packages installed automatically
✅ Users created
✅ Apache web server deployed
✅ Production-ready Ansible structure

---

# 🔥 BONUS (INTERVIEW POINTS)

- Roles = reusable + scalable architecture
- `loop` replaces `with_items`
- AL2023 uses `dnf`
- Separation of concerns (pkgs, users, webserver)
- Master playbook controls everything


---
# 🧹 STEP 11: Uninstall Everything (Bulk Change in Playbooks)

Sometimes you want to **remove all installed packages/users/services** using the same playbooks.

Instead of manually editing each file ❌
We use **automation (sed + find)** ✅

---

## 🔁 🔹 Method 1: Modify Single File

Example (for one file):

```bash
sed -i 's/present/absent/g' roles/pkgs/tasks/main.yml
```

### 📌 What it does:

- `present` ➝ `absent`
- Converts **install → uninstall**

---

## 🔁 🔹 Method 2: Modify ALL Playbooks (Recommended 🚀)

```bash
find . -type f -name "*.yml" -exec sed -i 's/present/absent/g' {} \;
```

---

## 🔍 Explanation (Important for Interview)

### 🔹 `find .`

- Search in **current directory**

### 🔹 `-type f`

- Select only **files** (not folders)

### 🔹 `-name "*.yml"`

- Target only **Ansible playbooks**

### 🔹 `-exec`

- Execute command on each file

### 🔹 `sed -i`

- Edit file **in-place**

### 🔹 `'s/present/absent/g'`

- Replace:
  - `present` ➝ `absent`
  - `g` = global (all occurrences)

### 🔹 `{}`

- Represents each file found

### 🔹 `\;`

- Ends the exec command

---

## ⚠️ Important Tip

Before running, check files:

```bash
find . -type f -name "*.yml"
```

---

## ▶️ STEP 12: Run Playbook (Uninstall Mode)

After replacing `present → absent`:

```bash
ansible-playbook -i hosts.ini master.yml
```

---

## ✅ Expected Result

All resources will be removed:

- ❌ Apache removed
- ❌ Nginx removed
- ❌ Docker removed
- ❌ Git removed
- ❌ Users removed

---

## 🔍 Verification

### Check packages removed:

```bash
ansible all -a "rpm -qa | grep httpd"
```

### Check users removed:

```bash
ansible all -a "cat /etc/passwd"
```

---

## ⚠️ Common Mistakes

❌ Wrong command:

```bash
find . -type f = -exec
```

✅ Correct:

```bash
find . -type f -exec sed -i 's/present/absent/g' {} \;
```

---

## 🚀 Pro Tip (Safe Execution)

Take backup before replacing:

```bash
cp -r roles roles_backup
```

---

# 🎯 Final Concept

This step shows:

✔ Idempotency in Ansible
✔ Bulk automation using Linux commands
✔ Real DevOps troubleshooting skills


---

# 🧹 STEP 13: Uninstall All Using Roles (Final Execution)

After modifying all role files:

```bash
present → absent
```

You can use the **same master playbook** to remove everything.

---

## ▶️ Run Master Playbook

```bash
ansible-playbook -i hosts.ini master.yml
```

---

## 🔄 What Happens Internally

Your roles were earlier doing:

```yaml
state: present
```

Now after replacement:

```yaml
state: absent
```

So Ansible will:

| Role      | Action                                        |
| --------- | --------------------------------------------- |
| pkgs      | Uninstall packages (git, docker, maven, etc.) |
| users     | Delete users                                  |
| webserver | Remove Apache (httpd)                         |

---

## 📦 Example (After Replacement)

### pkgs role:

```yaml
- name: Install Packages
  dnf:
    name: "{{ item }}"
    state: absent
```

---

### users role:

```yaml
- name: Create Users
  user:
    name: "{{ item }}"
    state: absent
```

---

### webserver role:

```yaml
- name: Install Apache
  dnf:
    name: httpd
    state: absent
```

---

## ✅ Expected Output

When you run:

```bash
ansible-playbook -i hosts.ini master.yml
```

You will see:

- Packages → **REMOVED**
- Users → **DELETED**
- Services → **STOPPED / REMOVED**

---

## 🔍 Verification

### Check packages removed:

```bash
ansible all -a "rpm -qa | grep httpd"
```

---

### Check users removed:

```bash
ansible all -a "cat /etc/passwd"
```

---

### Check service status:

```bash
ansible all -a "systemctl status httpd"
```

---

## ⚠️ Important Note

Ansible is **idempotent**, meaning:

- If already removed → no changes
- Safe to run multiple times

---

## 🚀 Real DevOps Insight

This pattern is used in industry for:

✔ Rollback deployments
✔ Cleanup infrastructure
✔ Environment reset
✔ Disaster recovery



---

# 🔐 STEP 14: Secure Secrets using Ansible Vault

Ansible Vault is used to **encrypt sensitive data** like:

- Database passwords
- API keys
- Credentials

---

## 📄 1. Create Encrypted File

```bash
ansible-vault create secret.txt
```

### 🔑 Enter Password:

```
New Vault password: root123
Confirm New Vault password: root123
```

---

## ✍️ Add Content Inside File

```ini
db_username=admin
db_password=hell@143
```

Save and exit (`ESC + :wq`)

---

## 🔍 2. View Encrypted File

```bash
cat secret.txt
```

### Output (Encrypted):

```
$ANSIBLE_VAULT;1.1;AES256
3132333435...
```

✔ Data is **fully encrypted**

---

## ✏️ 3. Edit Encrypted File

```bash
ansible-vault edit secret.txt
```

Enter password:

```
Vault password: root123
```

✔ File opens in decrypted mode for editing

---

## 🔓 4. Decrypt File

```bash
ansible-vault decrypt secret.txt
```

Now check:

```bash
cat secret.txt
```

### Output:

```ini
db_username=admin
db_password=hell@143
```

---

## 🔒 5. Encrypt Again

```bash
ansible-vault encrypt secret.txt
```

---

# 🔐 Using Vault in Playbook (IMPORTANT)

---

## 📄 Example Playbook with Vault

```yaml
---
- name: Use Vault Variables
  hosts: all
  vars_files:
    - secret.txt

  tasks:
    - name: Print DB Username
      debug:
        msg: "{{ db_username }}"
```

---

## ▶️ Run Playbook with Vault

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

---

# 🔑 Alternative (Better Practice)

Use password file:

```bash
echo "root123" > vault_pass.txt
chmod 600 vault_pass.txt
```

Run:

```bash
ansible-playbook playbook.yml --vault-password-file vault_pass.txt
```

---

# ⚠️ Common Mistakes

❌ Wrong command:

```bash
ansible-vault edit secret .txt
```

✅ Correct:

```bash
ansible-vault edit secret.txt
```

---

❌ Wrong spacing:

```bash
ansible-vault decrypt secret .txt
```

✅ Correct:

```bash
ansible-vault decrypt secret.txt
```


---

# 🎯 Final Outcome

You now know:

✅ Encrypt sensitive data
✅ Edit securely
✅ Use secrets in playbooks
✅ Follow real DevOps security practices


---

# 👀 STEP 15: View Encrypted File (Without Decrypting)

If you want to **read the contents of an encrypted file without decrypting it permanently**, use:

```bash
ansible-vault view secret.txt
```

---

## 🔑 Enter Password

```text
Vault password: root123
```

---

## 📄 Output (Decrypted View)

```ini
db_username=admin
db_password=hell@143
```

✔ File is **NOT decrypted on disk**
✔ It is only **temporarily shown in memory**

---

## 🔍 When to Use `view`

Use this command when:

- You just want to **check secrets quickly**
- You don’t want to **expose the file permanently**
- You are debugging a playbook

---

## 🔄 Comparison of Vault Commands

| Command                 | Purpose                 |
| ----------------------- | ----------------------- |
| `ansible-vault create`  | Create encrypted file   |
| `ansible-vault edit`    | Edit securely           |
| `ansible-vault view`    | View without decrypting |
| `ansible-vault decrypt` | Permanently decrypt     |
| `ansible-vault encrypt` | Encrypt file            |

---

## ⚠️ Important Note

- `view` is **safer than decrypt**
- File remains encrypted after viewing

---

## 🚀 Pro Tip

Instead of exposing secrets:

```bash
ansible-vault view secret.txt
```

❌ Avoid:

```bash
ansible-vault decrypt secret.txt
```

(only use decrypt when really needed)

---



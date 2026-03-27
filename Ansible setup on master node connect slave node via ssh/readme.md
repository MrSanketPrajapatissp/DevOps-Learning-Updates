Here is your **final professional `README.md`** 🔥
(With all issues, fixes, and your real practical flow included)

---

# 📘 Ansible Practical Setup Guide (Amazon Linux 2023)

---

## 🚀 Introduction

**Ansible** is an IT automation and configuration management tool used to:

- Automate server configuration
- Install packages
- Manage services 
- Deploy applications

👉 It is **agentless** and uses **SSH for communication**

---

## 🏗️ Architecture

We created **5 EC2 instances**:

| Role        | Server        |
| ----------- | ------------- |
| Master      | AnsibleMaster |
| Production  | pod1, pod2    |
| Development | dev1, dev2    |

---

## ⚙️ Step 1: Install Ansible (Amazon Linux 2023)

### ❌ Issue Faced

```bash
amazon-linux-extras: command not found
```

### 💡 Reason

Amazon Linux 2023 does not support `amazon-linux-extras`

---

### ✅ Fix

```bash
dnf update -y
dnf install ansible -y
ansible --version
```

---

## 📸 Screenshots



<img width="1920" height="1080" alt="Screenshot (166)" src="https://github.com/user-attachments/assets/700aaa24-fdba-4948-815a-a6f863bff1a9" />
<img width="1920" height="1080" alt="Screenshot (164)" src="https://github.com/user-attachments/assets/a0e3dcf4-c0fa-4339-aa49-da8fcd226c15" />

- Ansible installation
- Version check

---

## ⚙️ Step 2: Install Python

```bash
dnf install python3 python3-pip -y
```

---

## ⚙️ Step 3: Ad-hoc Commands (Localhost)

```bash
ansible -m ping localhost
ansible localhost -a "yum install git -y"
ansible localhost -a "mvn --version"
```

---

## ⚙️ Step 4: Server Configuration

### Set Hostname

```bash
hostnamectl set-hostname ansible-master
hostnamectl set-hostname pod1
hostnamectl set-hostname pod2
hostnamectl set-hostname dev1
hostnamectl set-hostname dev2
```

---

## 🔐 Step 5: Enable Root Login

```bash
vi /etc/ssh/sshd_config
```

### Changes:

```bash
PermitRootLogin yes
PasswordAuthentication yes
```

---

### ❌ Issue Faced

```bash
sshd.service failed (status=255)
```

### 💡 Cause

Invalid line:

```bash
Authentication:
```

---

### ✅ Fix

- Removed invalid line
- Validated config:

```bash
sshd -t
```

- Restarted service:

```bash
systemctl restart sshd
```

---

## ⚙️ Step 6: SSH Key Setup

### Generate Key (on AnsibleMaster)

```bash
ssh-keygen
```

---

### Copy Key to Nodes

```bash
ssh-copy-id root@<IP>
```

---

### ❌ Issue Faced

```bash
ssh-copy-id: No identities found
```

### 💡 Cause

Command executed on worker node instead of master

---

### ✅ Fix

Run `ssh-copy-id` only from **AnsibleMaster**

---

### ❌ Issue Faced

```bash
Permission denied
```

### 💡 Cause

- Root login disabled
- Password authentication disabled

---

### ✅ Fix

On each node:

```bash
passwd root
vi /etc/ssh/sshd_config
```

Ensure:

```bash
PermitRootLogin yes
PasswordAuthentication yes
```

---

## ⚙️ Step 7: Inventory File

### ❌ Issue Faced

```bash
/etc/ansible/hosts not found
```

---

### ✅ Fix

```bash
mkdir -p /etc/ansible
vi /etc/ansible/hosts
```

---

### Inventory Example

```ini
[prod]
172.31.45.233
172.31.45.44

[dev]
172.31.34.181
172.31.38.233
```

---

### Validate Inventory

```bash
ansible-inventory --list
```

---

## ⚙️ Step 8: Test Connectivity

```bash
ansible -m ping all
```

---

### ❌ Issue Faced

```bash
UNREACHABLE! Permission denied
```

---

### 💡 Cause

SSH key not copied to all nodes

---

### ✅ Fix

```bash
ssh-copy-id root@<all-nodes>
```

---

## ⚙️ Step 9: Final Output

```bash
SUCCESS => {
  "ping": "pong"
}
```

---

## 📸 Screenshots

(Add your screenshots)

- Inventory output
- Ansible ping success
- SSH login success

---

## ⚠️ Warnings Observed

```bash
discovered_interpreter_python
```

👉 This is normal and can be ignored

---

## 🧠 Key Learnings

- Amazon Linux 2023 uses `dnf`, not `yum-extras`
- SSH configuration must be correct
- Always validate with:

```bash
sshd -t
```

- Ansible requires **passwordless SSH**
- Always test manual SSH before Ansible

---

## 🔥 Troubleshooting Summary

| Issue                         | Cause               | Fix               |
| ----------------------------- | ------------------- | ----------------- |
| amazon-linux-extras not found | AL2023              | Use dnf           |
| sshd failed                   | Invalid config      | Remove wrong line |
| Permission denied             | Root login disabled | Enable root login |
| ssh-copy-id failed            | Wrong node          | Run from master   |
| Inventory missing             | Not created         | Create manually   |

---

## 🎯 Conclusion

✔ Installed Ansible
✔ Configured multiple nodes
✔ Enabled SSH access
✔ Created inventory
✔ Verified connectivity

---

# 🚀 Next Steps

- Ansible Playbooks (YAML)
- Automate package installation
- Deploy applications
- Integrate with CI/CD

---

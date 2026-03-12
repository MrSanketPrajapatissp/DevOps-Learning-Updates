# 🚀 Jenkins to EC2 File Transfer using Passwordless SSH

## 📌 Project Overview

This project demonstrates how to **transfer files from a Jenkins server to a remote test server using SSH key authentication**.

The setup uses **two Amazon EC2 instances**:

* Jenkins Server
* Test Server (Amazon Linux 2)

The Jenkins job automatically transfers files from the Jenkins workspace to the remote server **without requiring a password**.

---

# 🏗 Architecture

```
Developer → Jenkins Job → SSH Authentication → Test Server
                │
                └── Transfers Files → Remote EC2 Test Server
```

---

# ⚙️ Infrastructure Setup

## EC2 Instances

1. Jenkins Server
2. Test Server (Amazon Linux 2)

---

# 🔐 Step 1: Generate SSH Keys on Jenkins Server

Generate SSH key pair:

```bash
ssh-keygen
```

Start SSH agent:

```bash
eval $(ssh-agent -s)
```

Add private key:

```bash
ssh-add ~/.ssh/id_rsa
```

Copy the **public key** to the test server:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ec2-user@<TEST_SERVER_PRIVATE_IP>
```

Test connection:

```bash
ssh ec2-user@<TEST_SERVER_PRIVATE_IP>
```

If successful, Jenkins can connect to the test server **without entering a password**.

---

# 🔌 Step 2: Install Jenkins Plugin

Navigate to:

```
Manage Jenkins → Manage Plugins → Available Plugins
```

Install:

**Publish Over SSH Plugin**

This plugin allows Jenkins to **transfer files and execute commands on remote servers via SSH**.

---

# 🖥 Step 3: Configure Remote Server in Jenkins

Go to:

```
Manage Jenkins → Configure System → Publish over SSH
```

Add server configuration:

| Field    | Value                              |
| -------- | ---------------------------------- |
| Name     | TestServer                         |
| Hostname | `<PRIVATE_IP>`                     |
| Username | ec2-user                           |
| Key      | Paste Jenkins `id_rsa` private key |

Click **Test Configuration**.

---

# 🧪 Step 4: Create Jenkins Job

Create job:

```
New Item → Freestyle Project
```

Add build step:

```
Send files or execute commands over SSH
```

Example configuration:

| Option           | Value                  |
| ---------------- | ---------------------- |
| Source files     | `files/*`              |
| Remove prefix    | `files`                |
| Remote directory | `/home/ec2-user/files` |

---

# ▶️ Step 5: Build the Job

Workspace created:

```
/var/lib/jenkins/workspace/remotefiles/
```

Create test files:

```bash
mkdir files
touch test.py
```

Run:

```
Build Now
```

---

# 📸 Screenshots

## Jenkins Build Success Console Output

![Jenkins Console Output](screenshots/jenkins-build-success.png)

Shows successful file transfer from Jenkins to the remote EC2 instance.

---

## AWS EC2 Instances (Jenkins Server & Test Server)

![AWS EC2 Instances]<img width="1920" height="1080" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/1859dd7d-9619-4dc0-a52d-7944b6feb47e" />


Two EC2 instances used in this project:

* **MyJenkinsServer**
* **MyTestServer**

Both instances are running and connected through SSH for file transfer.

---

# ✅ Result

Files are automatically transferred:

```
Jenkins Server → Test Server
```

Destination:

```
/home/ec2-user/files
```

---

# 📚 Key Learnings

* Jenkins automation
* SSH key-based authentication
* Secure file transfer using Jenkins
* Basic CI/CD deployment workflow

---

# 🛠 Technologies Used

* Jenkins
* Amazon EC2
* Linux
* SSH
* Publish Over SSH Plugin

---

# 🚀 Future Improvements

* Convert Freestyle job to **Jenkins Pipeline**
* Integrate **GitHub repository**
* Build a **complete CI/CD pipeline**
* Add automated deployment stages

---

# 👨‍💻 Author

DevOps Learning Project – Jenkins & AWS

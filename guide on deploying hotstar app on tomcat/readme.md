# 🚀 End-to-End CI/CD: Java Web App (Hotstar Clone) using Jenkins, Tomcat & AWS S3

---

# 📌 Project Objective

This project demonstrates how to:

✔ Build a Java web application using Maven
✔ Automate build using Jenkins
✔ Deploy WAR file to Tomcat
✔ Store artifacts in AWS S3
✔ Debug real-world DevOps issues

---

# 🏗️ Architecture

```bash
GitHub → Jenkins → Maven Build → WAR → Tomcat → Browser
                               ↓
                              S3
```

---

# ⚙️ Prerequisites

* AWS EC2 instance (Amazon Linux / RHEL)
* Java 17 installed
* Jenkins installed
* Tomcat installed
* GitHub repository

---

# 🧰 Step 1: Install Required Tools

```bash
yum install git maven java-17-openjdk -y
```

Verify:

```bash
java -version
mvn -version
git --version
```

---

# 🐙 Step 2: Clone Repository

```bash
git clone https://github.com/MrSanketPrajapatissp/java-project-maven-new.git
```

---

# 🔧 Step 3: Setup Jenkins

Access Jenkins:

```bash
http://<EC2-IP>:8080
```

### 📸 Screenshot

👉 *(Add Jenkins Dashboard Screenshot here)*

---

# 🏗️ Step 4: Create Jenkins Job

* Click **New Item**
* Select **Freestyle Project**
* Name: `NewJob`

---

## 🔗 Configure Git

* Repository URL:

```bash
https://github.com/MrSanketPrajapatissp/java-project-maven-new.git
```

### 📸 Screenshot

👉 *(Add Git config screenshot)*

---

## 🔨 Build Step

Add:

```bash
mvn clean package
```

### 📸 Screenshot

👉 *(Add build step screenshot)*

---

# 📦 Step 5: Build Output

After build:

```bash
target/myapp.war
```

---

# ⚠️ Issue #1: Tomcat Deployment Failed (403 Error)

## ❌ Error

```bash
HTTP 403 Access Denied
```

---

## 🔍 Root Cause

* No user configured in Tomcat
* IP restriction enabled
* Wrong path used

---

# 🛠️ Step 6: Fix Tomcat Configuration

---

## 📁 Correct Path

```bash
/root/apache-tomcat-11.0.18
```

---

## ✏️ Edit Users File

```bash
vi /root/apache-tomcat-11.0.18/conf/tomcat-users.xml
```

Add:

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="admin123" roles="manager-gui,manager-script"/>
```

---

## ⚠️ Important Mistake (Real Issue Faced)

❌ Wrong:

```xml
manager-
script
```

✔ Fix:

```xml
manager-script
```

---

## 🔐 Remove IP Restriction

```bash
vi /root/apache-tomcat-11.0.18/webapps/manager/META-INF/context.xml
```

Comment:

```xml
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1" />
-->
```

---

## 🔄 Restart Tomcat

```bash
cd /root/apache-tomcat-11.0.18/bin
sh shutdown.sh
sh startup.sh
```

---

# 🌐 Step 7: Access Application

```bash
http://<EC2-IP>:8080/Mywebapp
```

### 📸 Screenshot

👉 *(Add Hotstar UI deployed screenshot)*

---

# ☁️ Step 8: Setup AWS S3

---

## 🪣 Create Bucket

* Name: `jenkins-artifact-bkt-sanket-user`
* Region: `ap-south-1`

---

## 🔐 Create IAM User

Permissions:

* S3 Full Access (or limited)

---

# 🔑 Step 9: Add AWS Credentials in Jenkins

Go to:
👉 Manage Jenkins → Credentials

Add:

* Kind: AWS Credentials
* Access Key
* Secret Key
* ID: `aws-s3-creds`

### 📸 Screenshot

👉 *(Add credentials screenshot)*

---

# ⚠️ Issue #2: S3 Profile Not Visible

## ❌ Problem

* Dropdown empty
* Credentials not showing

---

## 🔍 Root Cause

* Used wrong plugin (S3 Publisher)
* Missing S3 profile configuration

---

# 🛠️ Step 10: Install Correct Plugin

Install:

✔ Artifact Manager on S3
✔ AWS Credentials Plugin

---

# ⚙️ Step 11: Configure S3 Profile

Go to:
👉 Manage Jenkins → System

Scroll down → **Amazon S3**

Add:

* Profile Name: `my-s3-profile`
* Bucket: `jenkins-artifact-bkt-sanket-user`
* Region: `ap-south-1`
* Credentials: `aws-s3-creds`

### 📸 Screenshot

👉 *(Add S3 profile config screenshot)*

---

# ⚠️ Issue #3: Confusion Between Plugins

| Plugin           | Status        |
| ---------------- | ------------- |
| S3 Publisher     | ❌ Old         |
| Artifact Manager | ✅ Recommended |

---

# 🛠️ Step 12: Archive Artifacts

In Jenkins Job:

Add:

```bash
target/*.war
```

✔ This automatically uploads to S3

---

# 📸 Screenshot Sections

Add your screenshots here:

---

## 🔹 Jenkins Plugins

👉 *(Add plugin screenshots)*

## 🔹 AWS Credentials

👉 *(Add credential screenshot)*

## 🔹 S3 Profile

👉 *(Add system config screenshot)*

## 🔹 Build Success

👉 *(Add Jenkins build success screenshot)*

## 🔹 Application Output

👉 *(Add Hotstar UI screenshot)*

---

# 🎯 Final Outcome

✔ CI/CD pipeline working
✔ WAR deployed to Tomcat
✔ Application accessible
✔ Artifact stored in S3

---

# 🧠 Key Learnings

* Always verify file paths
* XML syntax errors break deployment
* Tomcat blocks remote access by default
* Plugin selection matters in Jenkins
* Modern approach > traditional plugins

---

# 🚀 Future Improvements

* Jenkins Pipeline (Jenkinsfile)
* Deploy from S3 → Tomcat
* Use IAM Roles instead of keys
* Dockerize application

---

# 👨‍💻 Author

**Sanket Prajapati**

---

# ⭐ Support

If this helped you:
👉 Give a ⭐ on GitHub

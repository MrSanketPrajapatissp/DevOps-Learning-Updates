---

# EC2 Deployment and Service Management (2026 Standards)

## ## 1. Prerequisites
* **Control Node:** A Jenkins server running on **Amazon Linux 2023**.
* **IAM Permissions:** An IAM user with `AmazonEC2FullAccess`.
* **Python Environment:** Python 3.11+ installed.

---

## ## 2. Installation & Configuration

In 2026, we avoid `sudo pip install` to prevent "Externally Managed Environment" errors. We use `dnf` for system-level dependencies.

### **Step A: Install Boto3 and Ansible Collections**

```bash
# Install python3-pip and boto3 (the modern version of boto)
sudo dnf install python3-pip -y
pip3 install boto3 botocore

# Install the updated AWS collection for Ansible
ansible-galaxy collection install amazon.aws
```

### **Step B: AWS Configuration**

Run this on your Jenkins server to securely store credentials:

```bash
aws configure
# Enter your Access Key ID and Secret Access Key when prompted
```

---

## ## 3. Execution: Provisioning & Management

### **A. Launching EC2 (The 2026 Way)**

The old `ec2` module is deprecated. Use `amazon.aws.ec2_instance`.

**File:** `ec2.yml`

```yaml
- name: Infrastructure Provisioning
  hosts: localhost
  connection: local
  tasks:
    - name: Launch 3 EC2 Instances
      amazon.aws.ec2_instance:
        name: "DevOps-Worker-Node"
        key_name: "your-key-pair"
        region: "ap-south-1"
        instance_type: "t2.micro"
        image_id: "ami-053b12d3152c0cc71" # Updated AL2023 AMI ID
        count: 3
        wait: yes
```

**Command:** `ansible-playbook ec2.yml`

### **B. Starting Tomcat via Tags**

To start the services specifically on your worker nodes, we use the `tags` attribute.

**File:** `tomcat.yml`

```yaml
- name: Manage Worker Node Services
  hosts: workernodes
  tasks:
    - name: Start the Tomcat service
      ansible.builtin.shell: "nohup /root/tomcat/bin/startup.sh &"
      tags: start_tomcat
```

**Command:** `ansible-playbook tomcat.yml --tags start_tomcat`

---

## ## 4. Troubleshooting: Upgradation Issues

| 2025 Legacy Issue                       | 2026 Modern Fix                                                                            |
| :-------------------------------------- | :----------------------------------------------------------------------------------------- |
| **`ImportError: No module named boto`** | Install `boto3`. The original `boto` does not support latest AWS features.                 |
| **`pip install` Error**                 | AL2023 protects system Python. Use `dnf install python3-pip` or `python -m venv`.          |
| **`ec2` module warning**                | Replace `ec2:` with `amazon.aws.ec2_instance:`.                                            |
| **Tomcat Shell Hang**                   | Use `nohup` combined with `&` to ensure the process detaches from the Ansible SSH session. |

---

## ## 5. Modern Implementation: The Senior DevOps Perspective

While your practical uses Ansible for provisioning, a **Senior DevOps Engineer in 2026** would apply the following "Real-World" logic:

### **Immutable Infrastructure (Terraform + Ansible)**

As your notes mention, we use **Terraform** for the "Outer Loop" (creating the VM) and **Ansible** for the "Inner Loop" (configuring the software inside).

- **Terraform** manages the state of the EC2 instances.
- **Ansible** is triggered via a Terraform `local-exec` provisioner only after the VM is ready.

### **AI Agent Integration**

- **Automated Remediation:** Instead of manually running `--tags start_tomcat`, modern setups use **AI-driven health checks**. If an AI agent (monitoring logs via OpenSearch or CloudWatch) detects Tomcat is down, it triggers the Ansible playbook automatically via a Jenkins API call.
- **LLM Script Refactoring:** Tools like **GitHub Copilot Workspace** are used to scan legacy `boto` scripts and automatically refactor them to `boto3` syntax, ensuring compatibility with the 2026 Python ecosystem.

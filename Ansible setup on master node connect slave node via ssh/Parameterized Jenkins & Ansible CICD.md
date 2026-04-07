#  Parameterized Jenkins & Ansible CI/CD (2026 Edition)

## ## 1. Prerequisites
* **Jenkins 2.400+** with the following plugins installed:
    * **Ansible Plugin**
    * **Git Plugin**
    * **Pipeline: Declarative Agent**
* **Control Node:** Jenkins server must have the `ansible` binary installed in its PATH.

---

## ## 2. Configuration: Jenkins UI Setup

To enable parameters, you must check **"This project is parameterized"** in the Jenkins job configuration.

### **A. String/Choice Parameter (Environment)**

- **Name:** `server`
- **Choices:**
  - `dev`
  - `test`
  - `prod`

### **B. String Parameter (Git Branch)**

- **Name:** `branch`
- **Default Value:** `main`
- **Choices:** `main`, `hotfix`, `master`

---

## ## 3. Execution: Modernized Pipeline Script

In 2026, we use **Declarative Pipelines** for better readability. We have replaced legacy hardcoded strings with `${params.variable}` syntax.

```groovy
pipeline {
    agent any

    parameters {
        choice(name: 'server', choices: ['dev', 'test', 'prod'], description: 'Select Target Environment')
        choice(name: 'branch', choices: ['main', 'hotfix', 'master'], description: 'Select Git Branch')
    }

    stages {
        stage('Checkout') {
            steps {
                // Modernized Git syntax using the branch parameter
                git branch: "${params.branch}",
                    url: 'https://github.com/ReyazShaik/java-project-maven-new.git'
            }
        }

        stage('Ansible Deploy') {
            steps {
                // Using the 'limit' parameter to target specific server groups
                ansiblePlaybook(
                    playbook: '/etc/ansible/deploy.yml',
                    inventory: '/etc/ansible/hosts',
                    credentialsId: 'tomcatcreds',
                    limit: "${params.server}",
                    disableHostKeyChecking: true,
                    installation: 'ansible'
                )
            }
        }
    }
}
```

---

## ## 4. Troubleshooting: Upgradation Issues

| Legacy Issue (2025)             | 2026 Modern Fix                                                                                           |
| :------------------------------ | :-------------------------------------------------------------------------------------------------------- |
| **Variable interpolation fail** | Use double quotes ""${params.var}"". Single quotes '${var}' treat the variable as literal text.         |
| **Branch not found**            | Ensure the AI or developer hasn't renamed `master` to `main` in the source repo; verify branch naming.    |
| **Inventory Mismatch**          | Ensure the groups in `/etc/ansible/hosts` (e.g., `[prod]`, `[dev]`) exactly match your choice parameters. |

---

## ## 5. Modern Implementation: AI-Enhanced DevOps

A Senior DevOps Engineer in 2026 uses AI to make parameters even more dynamic:

### **AI-Generated Dynamic Inventories**

Instead of a static choice parameter, engineers use **AI Agents** that query AWS/GCP APIs to see which instances are currently tagged as "healthy" and automatically populate the Jenkins choice list.

### **GitHub Copilot Workspace Integration**

- **Refactoring:** Copilot can take an old Scripted Pipeline and automatically convert it to this modern **Declarative** syntax.
- **Security Scanning:** AI agents scan your `deploy.yml` for hardcoded secrets or insecure vault paths and suggest using **HashiCorp Vault** or Jenkins Credentials binding instead of plain text passwords.

### **Automated Branch Validation**

AI-driven pipelines now perform a "Pre-flight check" on the selected `${params.branch}`. If the AI detects a build failure in that branch's history, it will block the manual deployment to `prod`.

---

> **Pro-Tip:** When using `${params.server}` in the `limit` field, always ensure your Ansible Inventory file is structured with headers like `[dev]`, `[test]`, and `[prod]` so the parameter maps correctly to your infrastructure groups.
════════════════════════════════════
  
  SECTION 1: STUDY NOTES

════════════════════════════════════

---
## Terraform Installation & Setup

### What It Is
Terraform is an Infrastructure as Code (IaC) tool that lets you create cloud resources using code instead of clicking through AWS console.

### Why It Exists
Manually creating cloud resources is slow, error-prone, and hard to replicate—Terraform automates and standardizes infrastructure deployment.

### Think of It Like This
Like a recipe for building houses—write it once, build identical houses anywhere, anytime.

### Real-World Scenario
DevOps teams use Terraform to spin up identical dev, staging, and production environments in minutes instead of hours.

### Key Points
- Installed via automated script (terrafomAL2.sh from GitHub)
- Requires initialization before first use (`terraform init`)
- Works with AWS via provider configuration
- Requires IAM role (Terra Role) attached to EC2 for AWS API access
- All code is written in HCL (HashiCorp Configuration Language)

### Commands / Config
```bash
# Setup
sh terr.sh
terraform init

# Install tree utility for directory visualization
yum install tree -y
tree
```

---
## Terraform Core Workflow Commands

### What It Is
The standard command sequence to validate, preview, create, and destroy infrastructure using Terraform.

### Why It Exists
Provides safety checks (validate, plan) before making actual changes to prevent costly mistakes in production.

### Think of It Like This
Like spell-check → print preview → print → shred for documents.

### Real-World Scenario
Engineers run `terraform plan` in team reviews to show exactly what will change before applying to production.

### Key Points
- `validate` checks syntax errors before execution
- `plan` shows what will be created/changed/destroyed (dry run)
- `apply` creates actual resources in AWS
- `destroy` removes all managed resources
- `--auto-approve` skips confirmation prompt (use carefully)

### Commands / Config
```bash
terraform validate
terraform plan
terraform apply --auto-approve
terraform destroy --auto-approve
```

---
## Provider Configuration

### What It Is
A provider tells Terraform which cloud platform (AWS, Azure, GCP) to use and how to connect to it.

### Why It Exists
Terraform supports multiple clouds—provider configuration specifies which one and its settings (region, credentials, version).

### Think of It Like This
Like choosing "English (US)" vs "English (UK)" in your phone settings—same language, different dialect.

### Real-World Scenario
Teams specify provider versions to prevent unexpected breaking changes when HashiCorp updates AWS provider.

### Key Points
- Provider block defines cloud platform
- Region specifies AWS geographic location (ap-south-1 = Mumbai)
- Version pinning ensures consistency across team
- Must run `terraform init` after adding/changing provider
- Stored in separate `provider.tf` for better organization

### Commands / Config
```hcl
# Simple provider
provider "aws" {
  region = "ap-south-1"
}

# Advanced provider with version
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.91.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

---
## Resource Blocks (EC2 Instance)

### What It Is
A resource block defines a cloud component (EC2, S3, VPC) that Terraform should create and manage.

### Why It Exists
Converts infrastructure requirements into declarative code—you describe what you want, Terraform figures out how to build it.

### Think of It Like This
Like ordering food—you say "one burger with cheese," the kitchen figures out how to make it.

### Real-World Scenario
DevOps creates standardized EC2 instances for app servers with consistent AMI, type, and tags across all environments.

### Key Points
- `resource "type" "name"` syntax required
- AMI ID specifies the OS image (region-specific)
- Instance type defines CPU/RAM (t2.micro = 1 vCPU, 1GB RAM)
- Tags help identify resources in AWS console
- Resource name used for referencing in outputs/dependencies

### Commands / Config
```hcl
resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"

  tags = {
    Name = "outputvarexample-server"
  }
}
```

---
## Output Values

### What It Is
Outputs display specific resource information (IP addresses, IDs, DNS) after Terraform creates resources.

### Why It Exists
You need resource details (like public IP) to connect to instances or pass values to other tools without manually checking AWS console.

### Think of It Like This
Like a receipt after shopping—shows you what you bought and important details (price, SKU).

### Real-World Scenario
CI/CD pipelines use output values to automatically configure deployed apps with database endpoints or load balancer URLs.

### Key Points
- Displayed after `terraform apply` completes
- Can output single values or lists
- Reference resource attributes using `resource_type.resource_name.attribute`
- Stored in separate `output.tf` for organization
- Use `[*]` for lists when using count parameter

### Commands / Config
```hcl
# Single instance outputs
output "instance-information" {
  value = [
    aws_instance.myinstance.public_ip,
    aws_instance.myinstance.private_ip,
    aws_instance.myinstance.public_dns
  ]
}

# Organized outputs in output.tf
output "instance_public_ip" {
  value = aws_instance.myinstance.public_ip
}

output "instance_id" {
  value = aws_instance.myinstance.id
}

output "instance_public_dns" {
  value = aws_instance.myinstance.public_dns
}

output "instance_arn" {
  value = aws_instance.myinstance.arn
}

# Multiple instances output
output "instance_id" {
  value = aws_instance.myinstance[*].id
}
```

---
## Variables (Input Variables)

### What It Is
Variables are placeholders that let you customize Terraform configurations without editing the main code.

### Why It Exists
Enables reusability—same code creates different environments (dev/staging/prod) by changing variable values.

### Think of It Like This
Like a form with blank fields—same form, different information filled in each time.

### Real-World Scenario
Teams use variables to deploy same infrastructure pattern across multiple AWS accounts with different instance sizes per environment.

### Key Points
- Defined in `variables.tf` file
- Values set in `terraform.tfvars` file
- Type can be string, number, bool, list, map
- Default value used if no value provided
- Description helps team understand variable purpose

### Commands / Config
```hcl
# variables.tf
variable "mybucket" {
  type        = string
  description = "This is my devops test bucket from TF"
  default     = ""
}

variable "instance_ami" {
  type    = string
  default = ""
}

variable "instance_type" {
  type    = string
  default = ""
}

variable "key_name" {
  type    = string
  default = ""
}

variable "instance_name" {
  type    = string
  default = ""
}

# terraform.tfvars
mybucket = "tf-example-reyaz-s3-bkt"

instance_ami   = "ami-08fe5144e4659a3b3"
instance_type  = "t2.micro"
key_name       = "530PMBATCH"
instance_name  = "MyinstanceB"

# Usage in main.tf
resource "aws_s3_bucket" "example" {
  bucket = var.mybucket
}

resource "aws_instance" "myinstance" {
  ami           = var.instance_ami
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = {
    Name = var.instance_name
  }
}
```

---
## S3 Bucket Resource

### What It Is
AWS S3 bucket resource definition in Terraform for creating object storage.

### Why It Exists
S3 buckets store files, backups, static websites—Terraform manages their creation and configuration as code.

### Think of It Like This
Like creating a Google Drive folder via code instead of clicking "New Folder."

### Real-World Scenario
DevOps creates S3 buckets for application logs, Terraform state files, or static website hosting with consistent naming conventions.

### Key Points
- Bucket name must be globally unique across all AWS
- Use variables for bucket name to avoid hardcoding
- Created with single resource block
- Verify creation in AWS S3 console after apply
- Destroyed completely with terraform destroy

### Commands / Config
```hcl
resource "aws_s3_bucket" "example" {
  bucket = var.mybucket
}

resource "aws_s3_bucket" "mys3bucket" {
  bucket = "test-bkt-lala-reya"
}
```

---
## Terraform State Management

### What It Is
Terraform tracks created resources in a state file to know what exists and what needs changing.

### Why It Exists
Without state tracking, Terraform can't tell if resources already exist or need updates—state is Terraform's memory.

### Think of It Like This
Like a shopping list you check off—you need it to remember what you've already bought.

### Real-World Scenario
Teams query state to see all managed resources, find specific resource IDs, or troubleshoot configuration drift.

### Key Points
- `terraform state list` shows all managed resources
- State file created automatically after first apply
- Contains sensitive data—never commit to Git
- Used to detect configuration drift
- Required for plan/apply operations

### Commands / Config
```bash
terraform state list
```

---
## Terraform Taint & Untaint

### What It Is
Taint marks a resource for destruction and recreation on next apply, even if configuration hasn't changed.

### Why It Exists
Forces replacement when resource is corrupted, misconfigured, or needs recreation without changing code.

### Think of It Like This
Like marking an email as "unread" to remind yourself to handle it again.

### Real-World Scenario
When an EC2 instance has configuration issues that can't be fixed in-place, taint it to force fresh deployment.

### Key Points
- `taint` marks resource for recreation
- `untaint` removes the mark before apply
- Affects only next apply—doesn't immediately destroy
- Apply shows tainted resource as destroy + create
- Useful for troubleshooting without code changes

### Commands / Config
```bash
# Mark resource for recreation
terraform taint aws_s3_bucket.mys3bucket
terraform taint aws_instance.myinstance

# Remove taint mark
terraform untaint aws_instance.myinstance

# Apply shows destroy+create for tainted resources
terraform apply --auto-approve
```

---
## Local Values (Locals)

### What It Is
Local values are computed values used multiple times within Terraform configuration to avoid repetition.

### Why It Exists
DRY principle (Don't Repeat Yourself)—define once, reference everywhere, easier to maintain and update.

### Think of It Like This
Like Excel cell references—change one cell, all formulas using it update automatically.

### Real-World Scenario
Define company naming convention once as local, apply to all resources—changing company name updates everything.

### Key Points
- Defined in `locals` block
- Reference with `local.variable_name` (singular)
- Can perform string interpolation with `${}`
- Computed at runtime, not user-input
- Great for tagging standards and naming patterns

### Commands / Config
```hcl
locals {
  project_name   = "My-Awesome-DevOps1"
  environment    = "Students"
  instance_count = 2

  tags = {
    Name = "${local.project_name}-${local.environment}"
  }
}

resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"
  count         = local.instance_count
  tags          = local.tags
}
```

---
## Count Parameter (Multiple Resources)

### What It Is
Count parameter creates multiple identical resources from a single resource block.

### Why It Exists
Avoids copy-pasting resource blocks—create 10 servers with one block instead of writing 10 blocks.

### Think of It Like This
Like a photocopy machine—one original, multiple identical copies.

### Real-World Scenario
Launch 5 identical web servers for load balancing without writing 5 separate resource blocks.

### Key Points
- `count = number` creates that many resources
- Each resource gets index [0], [1], [2]...
- Access specific instance with `resource_name[index]`
- Use `[*]` in outputs to get all instances' values
- Can use with local values for dynamic counts

### Commands / Config
```hcl
resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"
  count         = local.instance_count  # Creates 2 instances
  tags          = local.tags
}

output "instance_id" {
  value = aws_instance.myinstance[*].id  # Shows all instance IDs
}
```

---
## File Organization Best Practices

### What It Is
Splitting Terraform code into separate files (provider.tf, variables.tf, main.tf, output.tf) for better organization.

### Why It Exists
Large single files are hard to navigate—separation by purpose makes code maintainable and team-friendly.

### Think of It Like This
Like organizing a kitchen—utensils in one drawer, spices in another, not everything in one pile.

### Real-World Scenario
Teams standardize file structure so any engineer can instantly find variable definitions or output configurations.

### Key Points
- `provider.tf` → cloud provider configuration
- `variables.tf` → input variable definitions
- `terraform.tfvars` → actual variable values
- `main.tf` → resource definitions
- `output.tf` → output value definitions

### Commands / Config
```bash
tree  # View file structure
```

**Standard Structure:**
```
.
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── main.tf
└── output.tf
```

---
## IAM Role Attachment (Terra Role)

### What It Is
Attaching an IAM role to the EC2 instance running Terraform to grant AWS API permissions.

### Why It Exists
Terraform needs permissions to create/modify/delete AWS resources—IAM role provides those permissions securely without hardcoding credentials.

### Think of It Like This
Like giving your assistant (Terraform) your company badge (IAM role) to access restricted areas (AWS services).

### Real-World Scenario
Production Terraform runs on EC2 with IAM role instead of storing AWS access keys in code—more secure and auditable.

### Key Points
- Terra Role must have appropriate AWS permissions
- Attached to EC2 instance via AWS console
- Eliminates need for hardcoded AWS credentials
- More secure than access key/secret key
- Required before running terraform plan/apply

### Commands / Config
**Manual Step:** Attach the Terra Role to EC2 instance via AWS Console → EC2 → Actions → Security → Modify IAM Role

---
## Workspace Cleanup Commands

### What It Is
Commands to clean Terraform working directory by removing files and clearing terminal output.

### Why It Exists
Keeps workspace tidy between exercises, removes old configurations, provides fresh start for new deployments.

### Think of It Like This
Like clearing your desk before starting a new project.

### Real-World Scenario
Before starting new Terraform configuration, engineers clear old files to avoid conflicts or confusion.

### Key Points
- `rm -rf *` removes all files (use carefully)
- `clear` clears terminal screen
- `ls -al` shows all files including hidden ones
- `rm -rf ter.sh` removes specific file
- Always verify current directory before rm -rf

### Commands / Config
```bash
rm -rf *
clear
ls -al
rm -rf ter.sh
```

════════════════════════════════════

SECTION 2: COMPLETE WORKFLOW TABLE

════════════════════════════════════

| Step | Action | Tool / Command |
|------|--------|----------------|
| 1 | Launch EC2 instance and connect | AWS Console + MobaXterm |
| 2 | Set hostname | `sethostname Terraform` |
| 3 | Get installation script from GitHub | GitHub (devopsScript by reayaz → terrafomAL2.sh) |
| 4 | Create and paste script in EC2 | `vi terr.sh` |
| 5 | Run installation script | `sh terr.sh` |
| 6 | Clean workspace | `rm -rf *` then `clear` |
| 7 | Check files | `ls -al` |
| 8 | Initialize Terraform | `terraform init` |
| 9 | Create provider configuration | `vi provider.tf` |
| 10 | Create main configuration | `vi main.tf` |
| 11 | Validate syntax | `terraform validate` |
| 12 | Attach Terra IAM Role to EC2 | AWS Console → EC2 → Modify IAM Role |
| 13 | Preview changes | `terraform plan` |
| 14 | Create resources | `terraform apply --auto-approve` |
| 15 | Verify in AWS Console | AWS Console → EC2/S3 |
| 16 | Create variables file | `vi variables.tf` |
| 17 | Set variable values | `vi terraform.tfvars` |
| 18 | Create outputs file | `vi output.tf` |
| 19 | Visualize file structure | `tree` |
| 20 | List managed resources | `terraform state list` |
| 21 | Mark resource for recreation (optional) | `terraform taint <resource>` |
| 22 | Remove taint mark (optional) | `terraform untaint <resource>` |
| 23 | Destroy infrastructure | `terraform destroy --auto-approve` |

════════════════════════════════════

SECTION 3: INTERVIEW PREPARATION

════════════════════════════════════

## Terraform Installation & Setup
In practice, we automate Terraform installation using pre-built scripts from trusted repos rather than manual installation. The Terra IAM role is critical—without it, Terraform can't talk to AWS APIs, and all your commands fail with permission errors. What most people miss is that `terraform init` must run after any provider changes because it downloads provider-specific plugins to `.terraform` directory.

## Terraform Core Workflow Commands
The reason this matters is the sequence prevents disasters—validate catches syntax errors, plan shows exactly what changes before money gets spent creating resources. In production, we never use `--auto-approve` on apply; we always review the plan output first. What most people miss is that `terraform plan` doesn't guarantee apply will succeed—plan checks desired state, but AWS could reject requests due to quota limits or permission issues during actual apply.

## Provider Configuration
In practice, we pin exact provider versions in production to prevent surprise breaking changes when HashiCorp releases updates. The region setting determines where resources physically exist—ap-south-1 is Mumbai, which matters for latency to Indian users and data residency compliance. What most people miss is that changing the provider version requires re-running `terraform init` to download the new provider binary.

## Resource Blocks (EC2 Instance)
The reason resource naming matters is Terraform uses it for dependency tracking and state management—change the name, Terraform thinks it's a different resource and tries to destroy/recreate. In practice, AMI IDs are region-specific, so the same Ubuntu image has different IDs in Mumbai vs Virginia. What most people miss is that tags are how you'll find resources in AWS console when you have hundreds of instances—good tagging is not optional.

## Output Values
In practice, outputs feed into automation pipelines—CI/CD scripts grab the instance IP from Terraform output and use it for deployment or testing. What most people miss is that outputs are stored in state file (which might be shared with team), so never output secrets like passwords or private keys. The `[*]` splat operator is essential when using count—it collects values from all instances into a list instead of showing just one.

## Variables (Input Variables)
The reason terraform.tfvars exists is separation of code from data—same .tf files work across dev/staging/prod by swapping different .tfvars files. In practice, we never commit terraform.tfvars to Git if it contains secrets or environment-specific values. What most people miss is the variable precedence order—environment variables override tfvars, which override defaults.

## S3 Bucket Resource
In practice, S3 bucket names must be globally unique across all AWS accounts worldwide, not just your account. The reason this matters is bucket naming becomes a challenge at scale—teams use prefixes with company name or random suffixes. What most people miss is that deleting a bucket in AWS requires it to be empty, so Terraform destroy might fail if bucket contains files.

## Terraform State Management
The reason state is critical is it's how Terraform knows what exists in AWS versus what's in your code—lose state file, Terraform can't manage existing resources. In practice, we store state in S3 with locking via DynamoDB for team collaboration, never locally. What most people miss is state contains plaintext secrets like database passwords, so it needs encryption and access controls.

## Terraform Taint & Untaint
In practice, taint is the "turn it off and on again" of Terraform—when an instance is corrupted or has mysterious issues, taint forces a fresh build. What most people miss is that taint only marks the resource; actual destruction happens on next apply, so you have time to untaint if you change your mind. The reason this exists is sometimes fixing in-place is harder than starting fresh, especially with stateless resources like EC2 instances.

## Local Values (Locals)
The reason locals exist is reducing repetition and errors—update project name in one place, it propagates everywhere automatically. In practice, we use locals for complex expressions or string manipulations that would clutter resource blocks if repeated. What most people miss is locals are computed, not inputs—they can reference variables and other locals, creating powerful composition patterns.

## Count Parameter (Multiple Resources)
In practice, count creates identical resources which is great for stateless web servers but problematic for unique resources like databases. What most people miss is that count creates a list indexed by number [0][1][2], so deleting middle element causes Terraform to destroy and recreate all following elements—for_each is better for stable sets. The reason count matters is it turns Terraform from a one-resource-at-a-time tool into infrastructure at scale.

## File Organization Best Practices
In practice, this structure is so standard that engineers opening your repo expect these exact filenames and get confused if you deviate. What most people miss is that Terraform loads all .tf files in directory alphabetically, so splitting is for human organization, not execution order. The reason we separate is team collaboration—different people edit variables vs resources, and clean separation reduces merge conflicts.

## IAM Role Attachment (Terra Role)
In practice, IAM roles are how AWS services authenticate to other AWS services without hardcoded credentials in code. The reason this matters for Terraform is security—roles are temporary credentials rotated automatically, unlike access keys which live forever and can leak. What most people miss is the role must include policies for every AWS service Terraform will manage, not just EC2—creating S3 buckets requires S3 permissions in the role.

## Workspace Cleanup Commands
In practice, `rm -rf *` is dangerous in production—one wrong directory and you delete state files, losing track of all infrastructure. What most people miss is that hidden files (starting with dot) aren't deleted by `*` wildcard, so `.terraform` directory and `.tfstate` files survive. The reason we clean between exercises is avoiding confusion from leftover files, but in real projects, you preserve history and use version control instead of deleting.

════════════════════════════════════

SECTION 4: FACULTY NOTES — 100% VERBATIM

════════════════════════════════════

## Initial Setup Instructions

Launch One ec2 instance and connect with mobaXterm → sethostname Terraform

Got to Github devopsScript by reayaz → terrafomAL2.sh

Copy the code and create the fle in the ec2-instance Paste it→ name as terr.sh

Run → sh terr.sh

rm -rf  *

clear

Ls -al 

```sh
terraform init
```

---

## First EC2 Instance with Output Example

vi main.tf

terraform init

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"

  tags = {
    Name = "outputvarexample-server"
  }
}

output "instance-information" {
  value = [
    aws_instance.myinstance.public_ip,
    aws_instance.myinstance.private_ip,
    aws_instance.myinstance.public_dns
  ]
}
```

terraform validate

Attach the Terra Role to Ec2 instance.

terraform plain

terraform apply –auto-approve

terraform destroy –auto-approve

---

## S3 Bucket with Variables Example

ls 

vi terraform.tfvars

```hcl
mybucket = "tf-example-reyaz-s3-bkt"
```

vi variables.tf

```hcl
variable "mybucket" {
  type        = string
  description = "This is my devops test bucket from TF"
  default     = ""
}
```

vi provider.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.91.0"
    }
  }
}

provider "aws" {
 region = "ap-south-1"
}
```

```bash
vi main.tf
```

```hcl
resource "aws_s3_bucket" "example" {
  bucket = var.mybucket
}
```

```bash
yum install tree -y
```

```bash
tree

rm -rf ter.sh
```

```bash
terraform validate

terraform apply -auto-approve
```

Go to S3 on aws console check the bkt that we just created.

---

## EC2 Instance with Complete Variable Configuration

vi terraform.tfvars 

```hcl
instance_ami   = "ami-08fe5144e4659a3b3"
instance_type  = "t2.micro"
key_name       = "530PMBATCH"
instance_name  = "MyinstanceB"
```

vi variable.tf

```hcl
variable "instance_ami" {
  type    = string
  default = ""
}

variable "instance_type" {
  type    = string
  default = ""
}

variable "key_name" {
  type    = string
  default = ""
}

variable "instance_name" {
  type    = string
  default = ""
}
```

vi main.tf
 
```hcl
resource "aws_instance" "myinstance" {
  ami           = var.instance_ami
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = {
    Name = var.instance_name
  }
}
```

```bash
vi terraform.tfvars

vi variables.tf

vi main.tf

tree

terraform validate

vi output.tf
```

---

## Output Configuration for EC2 Instance

vi output.tf 

```hcl
output "instance_public_ip" {
  value = aws_instance.myinstance.public_ip
}

output "instance_id" {
  value = aws_instance.myinstance.id
}

output "instance_public_dns" {
  value = aws_instance.myinstance.public_dns
}

output "instance_arn" {
  value = aws_instance.myinstance.arn
}
```

tree

cat provider.tf

terraform apply  - - auto-approve

terraform destory - - auto-approve

---

## Taint and Untaint Example

vi  main.tf 

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"

  tags = {
    Name = "taint-server-example"
  }
}

resource "aws_s3_bucket" "mys3bucket" {
  bucket = "test-bkt-lala-reya"
}
```

terraform appy --auto-approve 

terraform state list

terraform taint aws_s3_bucket.mys3bucket

```bash
terraform taint aws_instance.myinstance

terraform untaint aws_instance.myinstance

terraform apply --auto-approve
```

```bash
terraform destroy --auto-approve
```

---

## Local Values and Count Example

vi main.tf

```hcl
locals {
  project_name   = "My-Awesome-DevOps1"
  environment    = "Students"
  instance_count = 2

  tags = {
    Name = "${local.project_name}-${local.environment}"
  }
}

resource "aws_instance" "myinstance" {
  ami           = "ami-08fe5144e4659a3b3"
  instance_type = "t2.micro"
  count         = local.instance_count
  tags          = local.tags
}

output "instance_id" {
  value = aws_instance.myinstance[*].id
}
```

terraform apply --auto-approve

terraform destroy --auto-approve

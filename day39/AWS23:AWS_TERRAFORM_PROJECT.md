![Progress](https://img.shields.io/badge/Progress-39%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 23 — Automating Infrastructure with Terraform

## 📝 Topic: Building a Scalable, Secure Web Server Architecture via Terraform
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla) with [Cloud Champ](https://github.com/)

**Date:** July 19, 2026

---

## 🎯 Learning Objectives
* Understand the guiding principle: manually configure it in the Console first, then automate it.
* Set up Terraform authentication and initialize a working directory.
* Define VPC, subnet, Internet Gateway, and Route Table resources in Terraform.
* Configure Security Groups for HTTP and SSH access.
* Deploy EC2 instances with `user_data` to auto-install and configure Apache.
* Set up an Application Load Balancer with Target Groups and health checks.
* Debug real Terraform plan/apply errors and practice proper cleanup with `terraform destroy`.

---

## 🧭 Part 1 — Project Overview & Guiding Principle

```
Goal: build a scalable, secure web server architecture,
      fully automated via Terraform

Core principle emphasized throughout:
  UNDERSTAND the manual Console configuration FIRST,
  THEN automate it with Terraform

  → Terraform isn't a shortcut around understanding
    AWS — it's an automation layer ON TOP of genuine
    understanding. Writing .tf files for a VPC/subnet/IGW
    setup you don't actually understand just produces
    working code you can't debug when it breaks.
```

> This directly validates the sequencing of this entire 30-day series — VPC (Day 04), Security Groups (Day 05), EC2 (Day 03), and ALB (Day 32.1) were all covered manually via the Console *before* this Terraform automation session. Today is genuinely the payoff of that ordering, not a standalone new topic.

---

## ⚙️ Part 2 — Environment Setup & Initialization

### Prerequisites

```
→ An AWS account
→ Terraform installed locally
```

### Authentication

```bash
aws configure
# (Reusing IAM user credentials + CLI setup from Day 10,
#  now consumed by Terraform's AWS provider instead of
#  the AWS CLI directly)
```

### Initialization

```bash
terraform init
```

```
What this does:
  → Initializes the working directory
  → Downloads and installs the necessary PROVIDERS
    (e.g., the AWS provider plugin)
  → Must be run before terraform plan/apply will work
```

---

## 🏗️ Part 3 — Defining Infrastructure Resources

### VPC & Networking

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block               = "10.0.1.0/24"
  availability_zone         = "us-east-1a"
  map_public_ip_on_launch   = true
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block               = "10.0.2.0/24"
  availability_zone         = "us-east-1b"
  map_public_ip_on_launch   = true
}
```

```
Custom VPC created + TWO subnets
  → Direct application of the VPC/subnet fundamentals
    from Day 04, now expressed as declarative code
```

> **Why `map_public_ip_on_launch` matters specifically:** without this argument set to `true`, instances launched in this subnet won't automatically receive a public IP — they'd be unreachable from the internet even with an Internet Gateway attached, because the INSTANCE itself never gets addressed publicly. This is a small, easy-to-forget argument with outsized consequences — exactly the class of "everything else configured correctly, one setting missing" bug seen repeatedly throughout this series.

### Connectivity: Internet Gateway & Route Tables

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
```

### Subnet Association

```hcl
resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public.id
}
```

> **Why this explicit association step can't be skipped:** creating a Route Table with a route to the Internet Gateway doesn't automatically apply it to any subnet. Without this association resource, the subnets remain on the VPC's DEFAULT route table (which has no internet route) — another direct echo of the Route Table mechanics first covered conceptually on Day 04.

---

## 🔒 Part 4 — Security & Instance Deployment

### Security Groups

```hcl
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

```
Ingress: HTTP (80) + SSH (22)
Egress: all traffic allowed out

→ Same Security Group rule structure from Day 05,
  now version-controlled as code instead of manually
  clicked through the Console
```

### S3 Bucket

```hcl
resource "aws_s3_bucket" "app_bucket" {
  bucket = "my-terraform-demo-bucket-2026"
}
```

### EC2 Instances with `user_data`

```hcl
resource "aws_instance" "web1" {
  ami                    = "ami-0abcdef1234567890"
  instance_type           = "t2.micro"
  subnet_id                = aws_subnet.public_a.id
  vpc_security_group_ids   = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              apt update -y
              apt install apache2 -y
              echo "<h1>Server 1</h1>" > /var/www/html/index.html
              systemctl start apache2
              EOF
}

resource "aws_instance" "web2" {
  ami                    = "ami-0abcdef1234567890"
  instance_type           = "t2.micro"
  subnet_id                = aws_subnet.public_b.id
  vpc_security_group_ids   = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              apt update -y
              apt install apache2 -y
              echo "<h1>Server 2</h1>" > /var/www/html/index.html
              systemctl start apache2
              EOF
}
```

> **`user_data` is the automation upgrade over the manual EC2/Apache deployment from Day 27:** instead of SSH-ing in and running `apt install httpd`/`systemctl start httpd` by hand, the entire bootstrap sequence runs automatically the moment the instance boots. Two instances, one in each subnet/AZ, directly implementing the high-availability principle from the Day 32.1 architecture project.

---

## ⚖️ Part 5 — Load Balancing & Finalization

### Application Load Balancer

```hcl
resource "aws_lb" "app_alb" {
  name               = "app-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_sg.id]
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id]
}
```

### Target Groups & Health Checks

```hcl
resource "aws_lb_target_group" "web_tg" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/"
    healthy_threshold   = 3
    unhealthy_threshold = 3
    interval            = 30
  }
}

resource "aws_lb_target_group_attachment" "web1" {
  target_group_arn = aws_lb_target_group.web_tg.arn
  target_id        = aws_instance.web1.id
  port             = 80
}

resource "aws_lb_target_group_attachment" "web2" {
  target_group_arn = aws_lb_target_group.web_tg.arn
  target_id        = aws_instance.web2.id
  port             = 80
}
```

### Listener Rules

```hcl
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app_alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}
```

```
Full traffic flow, now entirely codified:
  User → ALB → Listener → Target Group → web1 / web2

→ The exact same architecture from Day 32.1's manual
  ALB + ASG project, but now completely reproducible
  via `terraform apply` instead of dozens of
  Console clicks
```

---

## 🐛 Part 6 — Key Takeaways & Best Practices

### Live Debugging

```
The session includes REAL Terraform errors and their fixes:
  → Reading `terraform plan` output carefully to spot
    configuration mismatches BEFORE applying
  → Common issues: referencing a resource before it's
    defined, missing required arguments, mismatched
    subnet/AZ pairings
```

> **Why live debugging is more valuable than a clean, error-free demo:** `terraform plan`'s output is genuinely dense, and learning to read it under real (not staged) conditions is the actual skill — knowing what a "will be created," "must be replaced," or dependency-cycle error actually means in practice.

### Cleanup

```bash
terraform destroy
```

```
ALWAYS run this after finishing a project
  → Tears down every resource Terraform created,
    in the correct dependency order
  → Avoids unexpected AWS charges — the exact same
    discipline emphasized for CloudFront (Day 19)
    and every EC2-based project throughout this series
```

### Documentation & Tooling

```
Essential resources:
  → Terraform official documentation (provider/resource references)
  → VS Code extensions (Terraform syntax highlighting,
    validation — same category of tooling as the
    YAML/AWS Toolkit extensions from Day 11's CFT work)
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **`terraform init`** | Initializes a working directory and installs required providers |
| **`terraform plan`** | Shows what changes Terraform will make before applying them |
| **`terraform apply`** | Executes the planned changes, creating/modifying/destroying resources |
| **`terraform destroy`** | Tears down every resource managed by the current Terraform configuration |
| **Provider** | The plugin (e.g., AWS) Terraform uses to interact with a specific platform's API |
| **`map_public_ip_on_launch`** | A subnet argument controlling whether instances launched there auto-receive a public IP |
| **`user_data`** | EC2 launch-time script automation, executed once on first boot |
| **Target Group Attachment** | Explicitly links specific EC2 instances to a Load Balancer's Target Group |

---

## 📂 Summary of Tasks
- ✅ Understood: The "manually configure first, then automate" principle underlying this entire project.
- ✅ Set up: AWS authentication via `aws configure` and initialized Terraform with `terraform init`.
- ✅ Defined: A custom VPC with two subnets, correctly setting `map_public_ip_on_launch`.
- ✅ Configured: An Internet Gateway, Route Table, and explicit Route Table Associations.
- ✅ Defined: A Security Group allowing HTTP (80) and SSH (22) ingress.
- ✅ Created: An S3 bucket and two EC2 instances using `user_data` to auto-install Apache.
- ✅ Set up: An Application Load Balancer, Target Group, health checks, and Listener.
- ✅ Debugged: Real `terraform plan`/`apply` errors as part of the live walkthrough.
- ✅ Practiced: Running `terraform destroy` for full, clean resource teardown.

---

## 💡 My Takeaway

This session is genuinely the payoff for the entire series' sequencing — nearly every resource block written today (VPC, subnet, IGW, Route Table, Security Group, EC2, ALB, Target Group) maps directly onto something manually clicked through the Console back on Days 03, 04, 05, and 32.1. Writing the `.tf` files felt less like learning new AWS concepts and more like translating already-solid mental models into a declarative, version-controlled format — which is exactly the "understand it manually first" principle stated at the top of the session, now proven out in practice rather than just asserted.

`user_data` is the piece I'm most excited to reuse directly — it closes the gap between the fully manual EC2/Apache deployment from Day 27 (SSH in, run commands by hand) and genuine infrastructure automation. Combined with `terraform destroy` for guaranteed clean teardown, this is a real template I can adapt for spinning up and tearing down disposable environments for my own portfolio projects without repeating the same manual Console steps every time.

The live debugging section was more valuable than a polished, error-free walkthrough would have been — reading a real `terraform plan` diff under actual failure conditions is the skill that transfers to my own future Terraform work, far more than watching a clean happy-path demo would have.

---


## 🔗 Resources
* [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
* [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/cli)
* [Terraform VS Code Extension](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)
* [Terraform `user_data` for EC2](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#user_data)
* [Application Load Balancer + Target Groups (Terraform)](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
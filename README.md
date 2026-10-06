# aws-ha-selfhealing-web-app
# Highly Available & Self-Healing Web Application on AWS

## 🚀 Project Overview
This project focuses on transitioning a single, vulnerable EC2 instance into a production-ready, fault-tolerant fleet. Developed as part of the TechPeak Lab curriculum, this infrastructure architecture achieves high availability across multiple availability zones and implements an automated self-healing mechanism.

## 🏗️ Architecture Design
[ Internet Users ] ──(HTTP Port 80)──> [ Application Load Balancer ] ──> [ Auto Scaling Group ] ──> [ EC2 Web Fleet (Multi-AZ) ]

## 🛠️ Core Learning Objectives Met
* **Automation via User Data:** Bootstrapped Apache web servers automatically at launch using advanced IMDSv2 configuration scripts.
* **Security Isolation:** Enforced strict least-privilege networking via Security Group chaining (ALB -> EC2 instance layer).
* **High Availability (HA):** Distributed compute instances dynamically across isolated fault domains (Availability Zones).
* **Self-Healing Infrastructure:** Configured automated replacement parameters using combined target health checks and ASG tracking scaling limits (Min: 2 | Desired: 2 | Max: 4).

---

## 📸 Step-by-Step Implementation & Proof of Work

### Phase 1: Network Security Isolation
1. Provisioned `cloud-class-alb-sg` to receive external public traffic on Port 80.
2. Built `cloud-class-ec2-sg` and configured its inbound rules to accept HTTP traffic **ONLY** from the ALB Security Group ID, establishing a secure chain.

> 💡 *Replace the text inside the brackets below with your security group screenshot file name:*
> `![Security Group Rules](YOUR_SCREENSHOT_NAME_HERE.png)`

### Phase 2: Compute Automation Template
1. Generated an EC2 **Launch Template** choosing Amazon Linux 2023.
2. Enforced secure IMDSv2 parameters by extending the **Metadata response hop limit** to `2`.
3. Embedded an automated bootstrap script within the **User Data** block to handle system updates, start the Apache service, and isolate unique metadata metrics.

```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl start httpd
systemctl enable httpd

# Retrieve Instance Metadata (IMDSv2)
TOKEN=$(curl -s -X PUT "http://169.254.169" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169)
AZ=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169)

echo "<h1>Welcome to Cloud Class!</h1><p>Served from Instance ID: <b>$INSTANCE_ID</b> in Zone: <b>$AZ</b></p>" > /var/www/html/index.html
```

> 💡 *Replace the text inside the brackets below with your launch template screenshot file name:*
> `![Launch Template View](YOUR_SCREENSHOT_NAME_HERE.png)`

### Phase 3: High Availability & Load Balancing Layer
1. Configured an instance-level **Target Group** (`cloud-class-web-tg`) targeting port 80 with health tracks resting on `/`.
2. Created an internet-facing **Application Load Balancer** across multiple public subnets.
3. Assembled the final **Auto Scaling Group (ASG)** tied to our Target Group and mapped running capacity checks directly to ELB.

> 💡 *Replace the text inside the brackets below with your target group or ASG status screenshot file name:*
> `![Target Group Dashboard](YOUR_SCREENSHOT_NAME_HERE.png)`

---

## 🧪 Verification & Chaos Engineering Testing

### Test 1: Load Balancing Verification
By opening the ALB DNS endpoint URL in the browser and hitting refresh repeatedly, traffic dynamically swapped between independent EC2 Instance IDs running in completely separate Availability Zones (e.g., `us-east-1a` and `us-east-1b`).

> 💡 *Replace the text inside the brackets below with your browser test screenshot file names:*
> `![Load Balancing Verification 1](YOUR_SCREENSHOT_NAME_HERE.png)`
> `![Load Balancing Verification 2](YOUR_SCREENSHOT_NAME_HERE.png)`

### Test 2: Simulated Server Outage & Self-Healing Proof
To rigorously test resilience, one of our active EC2 instances was **manually terminated** in the console. 
* The ALB instantly recognized the drop and shifted 100% of user traffic onto the surviving instance with **zero user downtime**.
* Within moments, the Auto Scaling Group triggered an automation sequence to provision a fresh instance—maintaining our target baseline of 2 healthy instances.

> 💡 *Replace the text inside the brackets below with your ASG activity log screenshot file name:*
> `![ASG Self Healing Proof](YOUR_SCREENSHOT_NAME_HERE.png)`

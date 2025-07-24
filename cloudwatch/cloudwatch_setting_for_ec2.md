Absolutely! Here's a refined, professional, and GitHub-friendly version of the setup steps, optimized for clarity and readability in a `README.md` or documentation page:

---

## 📘 Setup: Monitor EC2 Custom Metrics with CloudWatch Agent

This guide outlines how to configure the **CloudWatch Agent** on an EC2 instance to collect **custom system metrics** like memory, disk, and swap usage. The setup uses **AWS Systems Manager (SSM)** for agent management and includes the use of **CollectD** for extended monitoring.

---

### 🚀 Prerequisites

* AWS account with administrative access
* EC2 instance (Amazon Linux 2 / Ubuntu preferred)
* SSM agent installed (enabled by default on Amazon Linux 2)
* Instance has internet access (via public subnet or NAT Gateway)

---

## 🛠️ Step-by-Step Instructions

### 🔐 Step 1: Create IAM Role for CloudWatch Agent

1. Go to **IAM > Roles > Create Role**
2. Select **EC2** as the trusted entity
3. Attach the following AWS-managed policies:

   * `AmazonSSMManagedInstanceCore`
   * `CloudWatchAgentServerPolicy`
4. Name the role (e.g., `EC2CloudWatchRole`) and create it

---

### ☁️ Step 2: Launch EC2 Instance with IAM Role

1. Launch a new EC2 instance
2. In the “Configure Instance” section, attach the IAM role created above
3. Ensure the instance is in a subnet with internet access (public or NAT)

---

### 🧩 Step 3: Register EC2 with Systems Manager

* Go to **Systems Manager > Fleet Manager > Managed Instances**
* Confirm your instance appears here (it means SSM is working correctly)
* If not:

  * Check IAM role and permissions
  * Check instance connectivity and security groups

---

### 🧰 Step 4: Install CloudWatch Agent & CollectD

Connect to the instance via **SSM Session Manager** or SSH, then:

```bash
# Amazon Linux / Ubuntu
sudo yum install -y amazon-cloudwatch-agent     # or `apt install` for Ubuntu
sudo yum install -y collectd
```

---

### ⚙️ Step 5: Configure the CloudWatch Agent

Run the built-in configuration wizard:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

The wizard will:

* Ask what metrics to monitor (CPU, memory, disk, etc.)
* Ask for the region and IAM credentials (choose **EC2 instance role**)
* Generate a JSON config file (typically at `/opt/aws/amazon-cloudwatch-agent/bin/config.json`)

---

### ▶️ Step 6: Start the CloudWatch Agent

Use the following command to apply the config and start the agent:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a start \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

---

### ✅ Step 7: Verify Metrics in CloudWatch

1. Open the AWS Management Console
2. Go to **CloudWatch > Metrics > CWAgent**
3. Look under **InstanceId** to find the metrics like:

   * `mem_used_percent`
   * `disk_used_percent`
   * `swap_used_percent`
   * `cpu_usage_idle`, etc.

---

## 🎯 Outcome

Your EC2 instance is now fully integrated with CloudWatch using the CloudWatch Agent and can push detailed system-level metrics for enhanced monitoring, dashboards, and alerts.

---

Let me know if you want to include:

* A shell script to automate all steps
* Terraform/CloudFormation templates
* Diagram for architecture overview

I'm happy to add it to your GitHub documentation.

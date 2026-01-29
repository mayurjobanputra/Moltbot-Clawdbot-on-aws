# ClawdBot AWS VPS Setup Guide

**Author:** AI-assisted setup documentation
**Last Updated:** 2026-01-28
**Status:** Phases 1 & 2 Complete

---

## 📖 Document Structure

This guide is divided into **two parts**:

| Part | What You Get | Who Needs It |
|------|--------------|--------------|
| **Part 1: Core Setup** (Phases 0-2) | Private Clawdbot server on AWS, secure and working | Everyone |
| **Part 2: Digital Twin** (Phases 3-5) | Full autonomous AI with email, GitHub, social accounts | Those building a VA/digital twin |

**After completing Part 1**, you have a fully functional Clawdbot accessible via:
- ✅ SSH tunnel + Gateway web chat (built-in)
- ✅ Private VPN like Tailscale (optional)
- ✅ `clawdbot chat` CLI command

**Part 2 is optional** — it adds email, GitHub, X/Twitter accounts so Clawdbot can act autonomously and communicate externally.

---

## Table of Contents

### Part 1: Core Clawdbot Setup (Required)
1. [Overview](#overview)
2. [Phase 0: AWS Account Setup](#phase-0-aws-account-setup-before-you-begin)
3. [Phase 1: AWS VPS Setup](#phase-1-aws-vps-setup)
4. [Phase 2: Security Lockdown](#phase-2-security-lockdown)

### Part 2: Digital Twin / VA Setup (Optional)
5. [Phase 3: Clawdbot Identity (Email + GitHub + X)](#phase-3-clawdbot-identity-email--github--x)
6. [Phase 4: Public Website (Hosted Separately)](#phase-4-public-website-hosted-separately)
7. [Phase 5: Crabwalk Integration](#phase-5-crabwalk-integration)

### Reference
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference Commands](#quick-reference-commands)
10. [Progress Tracking](#progress-tracking)

---

# PART 1: CORE CLAWDBOT SETUP

> **Goal:** Get Clawdbot running privately on AWS with secure access.
> **Time:** ~30-60 minutes
> **Cost:** ~$19/month (or free with AWS Activate credits)

---

## Overview

### What Part 1 Gives You
- ✅ Clawdbot running 24/7 on your own AWS server
- ✅ Secure access via SSH tunnel (only you can connect)
- ✅ Built-in web chat interface
- ✅ CLI access (`clawdbot chat`)
- ✅ Your AWS Bedrock credits powering Claude models

### How You'll Access Clawdbot

After Part 1, you have several access methods:

| Method | Security | Setup |
|--------|----------|-------|
| **SSH tunnel + Web chat** | ✅ Excellent | Automatic with `ssh clawdbot` |
| **CLI** (`ssh clawdbot`, then `clawdbot chat`) | ✅ Excellent | Built-in |
| **Tailscale VPN** | ✅ Excellent | Optional, ~5 min setup |
| **VS Code + Roo Code** | ✅ Excellent | Connect via SSH, AI in your editor |

**No public exposure needed for basic use!**

### ⚠️ Important: Keep This Server Private

**Do NOT run a public website on this EC2 instance.** The Clawdbot server should remain completely private for security reasons.

If you want a public website, host it separately (Vercel, Netlify, etc.) — see Part 2.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DUAL AI ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────┐     ┌─────────────────────────────────┐    │
│  │  PRIVATE EC2 (This Server)  │     │  PUBLIC (Vercel/Netlify/etc)    │    │
│  │  ════════════════════════   │     │  ════════════════════════════   │    │
│  │                             │     │                                 │    │
│  │  🦞 Clawdbot Gateway        │     │  🌐 Your Website                │    │
│  │  • Full AI capabilities     │     │  • Public chatbot (stateless)   │    │
│  │  • Shell access, tools      │     │  • Scheduling integration       │    │
│  │  • Memory & workspace       │     │  • Contact forms                │    │
│  │  • GitHub push access       │     │  • Direct API calls only        │    │
│  │  • ONLY YOU can access      │     │  • Anyone can access            │    │
│  │                             │     │                                 │    │
│  │  Access: SSH tunnel only    │     │  Access: HTTPS (public)         │    │
│  │  Port 18789: localhost      │     │                                 │    │
│  └──────────────┬──────────────┘     └─────────────────────────────────┘    │
│                 │                                                            │
│                 │  Clawdbot updates website via:                             │
│                 │  • Git push (using its own GitHub account)                 │
│                 │  • S3 sync                                                 │
│                 ▼                                                            │
│  ┌─────────────────────────────┐                                            │
│  │  📧 Clawdbot's Identity     │                                            │
│  │  • Email: ai@yourdomain.com │                                            │
│  │  • GitHub: yourdomain-ai    │                                            │
│  │  • Can receive alerts       │                                            │
│  │  • Can push to repos        │                                            │
│  └─────────────────────────────┘                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Why separate systems?**

| Concern | Private Clawdbot | Public Website |
|---------|------------------|----------------|
| **Security** | Full tool access = high risk if exposed | Stateless = low risk |
| **Attack surface** | SSH tunnel only | Standard web security |
| **Capabilities** | Shell, files, memory, GitHub | Text generation only |
| **Identity** | Your digital twin (full power) | Public receptionist |

### ⚠️ CRITICAL SECURITY WARNING (from @0xSammy)

> **923 ClawdBot gateways were found exposed with ZERO auth!**
> 
> - Shell access, browser automation, API keys all exposed
> - Anyone can connect to your IP and gain FULL control of your device
> 
> **THE FIX:** Change `bind: "all"` to `bind: "loopback"` and restart!

---

## 💡 How to Access ClawdBot Dashboard

Since ClawdBot is bound to localhost for security, you need an SSH tunnel to access it.

### Quick Access (Recommended)

```bash
# Start tunnel in background (keeps running even if terminal closes)
ssh -L 18789:127.0.0.1:18789 -N clawdbot &

# Open dashboard
open "http://localhost:18789/?token=YOUR_GATEWAY_TOKEN"
```

### Managing the SSH Tunnel

**Start tunnel (foreground - stays in terminal):**
```bash
ssh clawdbot
# This also opens an interactive shell on EC2
```

**Start tunnel (background - runs silently):**
```bash
ssh -L 18789:127.0.0.1:18789 -N clawdbot &
```

**Check if tunnel is running:**
```bash
ps aux | grep "ssh.*18789" | grep -v grep
# Or check if port is listening:
lsof -i :18789
```

**Stop the tunnel:**
```bash
# If running in foreground: Ctrl+C
# If running in background:
pkill -f "ssh.*18789.*clawdbot"
# Or find the PID and kill it:
ps aux | grep "ssh.*18789" | grep -v grep
kill <PID>
```

**Restart tunnel (if "Address already in use" error):**
```bash
# Kill existing tunnel first
pkill -f "ssh.*18789"
# Wait a moment
sleep 2
# Start fresh
ssh -L 18789:127.0.0.1:18789 -N clawdbot &
```

### Dashboard URL

```
http://localhost:18789/?token=YOUR_GATEWAY_TOKEN
```

**Get your token** (on EC2):
```bash
cat ~/.clawdbot/clawdbot.json | grep '"token"'
```

---

## Phase 0: AWS Account Setup (Before You Begin)

### Step 0.1: Create an AWS Account

If you don't have an AWS account yet:

1. Go to [aws.amazon.com](https://aws.amazon.com/)
2. Click "Create an AWS Account"
3. Provide email, password, and account name
4. Add payment information (credit card required)
5. Verify your identity (phone verification)
6. Choose support plan (Basic - Free is fine)

**💡 Cost Expectations:**

| Instance Type | RAM | Monthly Cost | Notes |
|---------------|-----|--------------|-------|
| `t2.micro` | 1GB | **Free** (Free Tier) | ⚠️ NOT enough for ClawdBot |
| `t2.small` | 2GB | ~$17/month | ✅ **Recommended minimum** |
| `t3.small` | 2GB | ~$15/month | Slightly cheaper, burstable |
| `t3.medium` | 4GB | ~$30/month | For heavy usage |

**Other costs:**
- Data transfer: First 100GB/month free
- Elastic IP: Free when attached to running instance
- EBS Storage (20GB): ~$1.60/month (30GB free in Free Tier)

**⚠️ IMPORTANT:** ClawdBot needs ~600MB+ RAM. The `t2.micro` (1GB) will crash with out-of-memory errors. Use **t2.small (2GB)** minimum!

**💰 AWS Activate Credits:** If you have $1,000 in credits, a t2.small costs ~$17/month × 12 = ~$200/year, so your credits last **5+ years**!

### Step 0.2: Optional - AWS Startups Program ($1,000 Credits!)

If you're building a startup or project, apply for **AWS Activate**:

🔗 **Apply here:** [aws.amazon.com/activate](https://aws.amazon.com/activate/)

**Benefits:**
- **$1,000 in AWS credits** (Founders tier)
- Valid for 2 years
- Works across ALL AWS services (EC2, S3, Lambda, etc.)
- Great for running ClawdBot long-term without worrying about costs

**Requirements:**
- Must be a new or early-stage startup
- Self-funded or raised less than seed funding
- Not previously received Activate credits

Even if you're just experimenting, it's worth applying!

### Step 0.3: Access AWS CloudShell

1. Log into [AWS Console](https://console.aws.amazon.com/)
2. Click the CloudShell icon (terminal icon in top navigation bar)
3. Wait for the environment to initialize
4. You're ready to run commands!

**Note:** CloudShell is free and comes pre-configured with AWS CLI.

---

## Phase 1: AWS VPS Setup

### Prerequisites
- ✅ AWS Account with payment method
- ✅ AWS CloudShell access (or AWS CLI configured locally)
- 🔑 Anthropic API key (or OpenAI key for Claude/GPT models)

### Step 1.1: Create Key Pair

Run in **CloudShell**:

```bash
# Create a key pair for SSH access
aws ec2 create-key-pair \
    --key-name clawdbot-key \
    --query 'KeyMaterial' \
    --output text > clawdbot-key.pem

# Set proper permissions
chmod 400 clawdbot-key.pem

# Verify it was created (should show BEGIN RSA PRIVATE KEY)
cat clawdbot-key.pem | head -3
```

- [ ] **CHECKPOINT:** Key pair created

### Step 1.2: Create Security Group

```bash
# Get your default VPC ID
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=is-default,Values=true" --query "Vpcs[0].VpcId" --output text)
echo "VPC ID: $VPC_ID"

# Create security group
SG_ID=$(aws ec2 create-security-group \
    --group-name clawdbot-sg \
    --description "Security group for ClawdBot VPS" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)
echo "Security Group ID: $SG_ID"

# Allow SSH (port 22)
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Allow HTTP (port 80) - For your future website
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Allow HTTPS (port 443) - For your future website
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# ⚠️ IMPORTANT: Verify rules were added
aws ec2 describe-security-groups \
    --group-ids $SG_ID \
    --query 'SecurityGroups[0].IpPermissions'

# NOTE: We are NOT opening port 18789 (ClawdBot gateway)
# This will ONLY be accessible via SSH tunnel
```

**⚠️ If security group rules are empty**, run the authorize commands again!

- [ ] **CHECKPOINT:** Security group created with ports 22, 80, 443 open

### Step 1.3: Launch EC2 Instance

```bash
# Get the latest Amazon Linux 2023 AMI
AMI_ID=$(aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=al2023-ami-*-x86_64" "Name=state,Values=available" \
    --query "sort_by(Images, &CreationDate)[-1].ImageId" \
    --output text)
echo "AMI ID: $AMI_ID"

# Launch instance with 20GB storage (default 8GB is NOT enough for ClawdBot!)
INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type t2.small \
    --key-name clawdbot-key \
    --security-group-ids $SG_ID \
    --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ClawdBot-VPS}]' \
    --query 'Instances[0].InstanceId' \
    --output text)
echo "Instance ID: $INSTANCE_ID"

# Wait for instance to be running (~30 seconds)
echo "Waiting for instance to start..."
aws ec2 wait instance-running --instance-ids $INSTANCE_ID
echo "✅ Instance is running!"

# Get the public IP
PUBLIC_IP=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' \
    --output text)
echo "🌐 Public IP: $PUBLIC_IP"

# Save this info for later
echo "INSTANCE_ID=$INSTANCE_ID" >> clawdbot-instance.env
echo "PUBLIC_IP=$PUBLIC_IP" >> clawdbot-instance.env
echo "SG_ID=$SG_ID" >> clawdbot-instance.env
```

- [ ] **CHECKPOINT:** EC2 instance launched with 20GB storage

### Step 1.4: Allocate Elastic IP (Permanent IP)

**⚠️ IMPORTANT:** Without an Elastic IP, your public IP will change every time the instance restarts!

```bash
# Allocate Elastic IP
ALLOC_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
echo "Allocation ID: $ALLOC_ID"

# Get the Elastic IP address
EIP=$(aws ec2 describe-addresses --allocation-ids $ALLOC_ID --query 'Addresses[0].PublicIp' --output text)
echo "Elastic IP: $EIP"

# Associate with your instance
aws ec2 associate-address --instance-id $INSTANCE_ID --allocation-id $ALLOC_ID

echo "✅ Your PERMANENT IP is now: $EIP"

# Update the saved IP
echo "ELASTIC_IP=$EIP" >> clawdbot-instance.env
```

**Note:** Elastic IPs are free when attached to a running instance. They cost ~$0.005/hour (~$3.60/month) only if the instance is stopped or the IP is unattached.

- [ ] **CHECKPOINT:** Elastic IP allocated and associated

### Step 1.5: Save PEM Key to Local Machine

In **CloudShell**, display the full key:
```bash
cat clawdbot-key.pem
```

Copy the **entire output** (from `-----BEGIN RSA PRIVATE KEY-----` to `-----END RSA PRIVATE KEY-----`).

**On your local machine**, choose ONE method:

**Option A: Using VS Code (easiest)**
```bash
code ~/.ssh/clawdbot-key.pem
# Paste key content, save with Cmd+S
chmod 400 ~/.ssh/clawdbot-key.pem
```

**Option B: Using cat with heredoc**
```bash
mkdir -p ~/.ssh
cat > ~/.ssh/clawdbot-key.pem << 'EOF'
-----BEGIN RSA PRIVATE KEY-----
[PASTE YOUR KEY HERE]
-----END RSA PRIVATE KEY-----
EOF
chmod 400 ~/.ssh/clawdbot-key.pem
```

**Option C: Using pbpaste (Mac only)**
Copy the key to clipboard, then:
```bash
mkdir -p ~/.ssh
pbpaste > ~/.ssh/clawdbot-key.pem
chmod 400 ~/.ssh/clawdbot-key.pem
```

**Verify:**
```bash
ls -la ~/.ssh/clawdbot-key.pem
# Should show: -r-------- (permissions 400)
```

- [ ] **CHECKPOINT:** PEM key saved locally with correct permissions

### Step 1.6: SSH into the Instance

From your **local terminal**:
```bash
# Replace <ELASTIC_IP> with your Elastic IP from above
ssh -i ~/.ssh/clawdbot-key.pem ec2-user@<ELASTIC_IP>
```

If prompted "Are you sure you want to continue connecting?", type `yes`.

**If connection hangs:** Wait 2 minutes (instance may still be booting), then try again.

- [ ] **CHECKPOINT:** Successfully SSH'd into instance

### Step 1.7: Install Node.js 22+ on EC2

Once SSH'd into the EC2 instance:
```bash
# Update system
sudo dnf update -y

# Install Node.js 22 (ClawdBot requires Node >= 22)
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo dnf install -y nodejs

# Verify installation
node --version  # Should show v22.x.x
npm --version
```

- [ ] **CHECKPOINT:** Node.js 22+ installed

### Step 1.8: Install ClawdBot

```bash
# Install ClawdBot CLI
curl -fsSL https://clawd.bot/install.sh | bash

# If install fails due to space, clean up first:
# npm cache clean --force
# rm -rf ~/.npm/_cacache

# Verify installation
clawdbot --version
```

- [ ] **CHECKPOINT:** ClawdBot installed

### Step 1.9: Run ClawdBot Onboarding

```bash
# Run the onboarding wizard
clawdbot onboard --install-daemon
```

**During onboarding, choose these options:**

| Prompt | Recommended Choice |
|--------|-------------------|
| Security warning | Yes (I understand) |
| Onboarding mode | **QuickStart** |
| Model/auth provider | Skip for now (add API key later) |
| Default model | Keep current |
| Select channel | Skip for now |
| Configure skills | **Skip for now** (most are Mac-only) |
| Enable hooks | Skip for now |
| Gateway service runtime | Node (default) |

**⚠️ QuickStart automatically sets:**
- Gateway bind: **Loopback (127.0.0.1)** ✅ SECURE!
- Gateway auth: Token (default)
- Tailscale: Off

- [ ] **CHECKPOINT:** ClawdBot onboarding completed

### Step 1.10: Start and Verify Gateway

```bash
# Start the gateway service
systemctl --user start clawdbot-gateway

# Enable auto-start on boot
systemctl --user enable clawdbot-gateway

# Check status
systemctl --user status clawdbot-gateway

# Check gateway health
clawdbot gateway status
clawdbot health
```

- [ ] **CHECKPOINT:** ClawdBot gateway is running

### Step 1.11: Configure AI Provider (AWS Bedrock)

You have two options for AI models. We'll use **AWS Bedrock** since you have an AWS account with credits!

#### Why AWS Bedrock?
- ✅ Uses your existing AWS credits ($1,000 Activate!)
- ✅ No separate Anthropic API key needed
- ✅ Pay-as-you-go pricing
- ✅ Claude models available (Sonnet, Haiku, Opus)

#### Step 1.11.1: Enable Bedrock Models in AWS Console

1. Go to [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/)
2. Select your region (e.g., `us-east-1` or `us-west-2`)
3. Click **"Model access"** in the left sidebar
4. Click **"Manage model access"**
5. Check the boxes for:
   - ✅ Anthropic - Claude 3 Sonnet
   - ✅ Anthropic - Claude 3 Haiku
   - ✅ Anthropic - Claude 3.5 Sonnet (if available)
6. Click **"Save changes"**
7. Wait for status to change to **"Access granted"** (usually instant)

- [ ] **CHECKPOINT:** Bedrock model access enabled

#### Step 1.11.2: Create IAM User for Bedrock Access

In **CloudShell**, run:

```bash
# Create a dedicated IAM user for ClawdBot
aws iam create-user --user-name clawdbot-bedrock

# Create access policy for Bedrock
cat > /tmp/bedrock-policy.json << 'EOF'
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "arn:aws:bedrock:*::foundation-model/anthropic.*"
        }
    ]
}
EOF

# Create the policy
aws iam create-policy \
    --policy-name ClawdBotBedrockAccess \
    --policy-document file:///tmp/bedrock-policy.json

# Get your AWS account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo "Account ID: $ACCOUNT_ID"

# Attach policy to user
aws iam attach-user-policy \
    --user-name clawdbot-bedrock \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/ClawdBotBedrockAccess

# Create access keys
aws iam create-access-key --user-name clawdbot-bedrock

# ⚠️ SAVE THE OUTPUT! You'll need:
# - AccessKeyId
# - SecretAccessKey
```

**⚠️ IMPORTANT:** Save the `AccessKeyId` and `SecretAccessKey` somewhere secure - you cannot retrieve the secret key again!

- [ ] **CHECKPOINT:** IAM user created with Bedrock access

#### Step 1.11.3: Configure AWS Credentials on EC2

SSH into your EC2 instance:
```bash
ssh clawdbot
```

**Option A: AWS Credentials File (for CLI testing)**

```bash
# Set up AWS credentials for ClawdBot
mkdir -p ~/.aws

cat > ~/.aws/credentials << 'EOF'
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID_HERE
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY_HERE
EOF

cat > ~/.aws/config << 'EOF'
[default]
region = us-east-1
output = json
EOF

# Set proper permissions
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config

# Verify credentials work
aws bedrock list-foundation-models --query 'modelSummaries[?contains(modelId, `anthropic`)].modelId' --region us-east-1
```

You should see a list of Anthropic models - that means it's working!

> ⚠️ **CRITICAL: Clawdbot Systemd Service Issue**
>
> The AWS credentials file works for CLI commands, but **Clawdbot runs as a systemd service** which does NOT inherit your shell environment or read `~/.aws/credentials` automatically.
>
> You MUST also complete **Option B** below for Clawdbot to access Bedrock!

**Option B: Systemd Drop-in Configuration (REQUIRED for Clawdbot)**

Clawdbot only supports Amazon Bedrock via the **AWS SDK default credential chain** (environment variables, shared config, instance roles). The systemd service runs in isolation and won't see your shell credentials.

Create a systemd drop-in file to inject AWS credentials into the service:

```bash
# Create drop-in directory
mkdir -p ~/.config/systemd/user/clawdbot-gateway.service.d

# Create environment file (replace with YOUR keys)
cat > ~/.config/systemd/user/clawdbot-gateway.service.d/aws.conf << 'EOF'
[Service]
Environment="AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID_HERE"
Environment="AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY_HERE"
Environment="AWS_REGION=us-east-1"
EOF

# Set secure permissions
chmod 600 ~/.config/systemd/user/clawdbot-gateway.service.d/aws.conf

# Reload systemd to pick up changes
systemctl --user daemon-reload

# Restart gateway
systemctl --user restart clawdbot-gateway
```

**Verify the drop-in is loaded:**
```bash
systemctl --user status clawdbot-gateway
# Look for "Drop-In:" section showing aws.conf
```

- [ ] **CHECKPOINT:** AWS credentials configured on EC2 (both file AND systemd)

#### Step 1.11.4: Configure ClawdBot for Bedrock

> ⚠️ **CRITICAL: Use `models.providers` configuration!**
>
> The `clawdbot configure` commands may not work for newer models. You MUST add the model definition under `models.providers` in `clawdbot.json` for cross-region inference profiles to work.

**Edit `~/.clawdbot/clawdbot.json` and add the `models.providers` section:**

```bash
# Install nano if not available
sudo dnf install nano -y

# Edit the config file
nano ~/.clawdbot/clawdbot.json
```

Add this `models` section (merge with existing config):

```json
{
  "models": {
    "providers": {
      "amazon-bedrock": {
        "baseUrl": "https://bedrock-runtime.us-east-1.amazonaws.com",
        "api": "bedrock-converse-stream",
        "auth": "aws-sdk",
        "models": [
          {
            "id": "us.anthropic.claude-opus-4-5-20251101-v1:0",
            "name": "Claude Opus 4.5 (US)",
            "reasoning": true,
            "input": ["text", "image"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
            "name": "Claude Sonnet 4.5 (US)",
            "reasoning": false,
            "input": ["text", "image"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "anthropic.claude-3-haiku-20240307-v1:0",
            "name": "Claude 3 Haiku",
            "reasoning": false,
            "input": ["text", "image"],
            "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
            "contextWindow": 200000,
            "maxTokens": 4096
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "amazon-bedrock/us.anthropic.claude-opus-4-5-20251101-v1:0",
        "fallbacks": ["amazon-bedrock/anthropic.claude-3-haiku-20240307-v1:0"]
      }
    }
  }
}
```

**Restart the gateway:**
```bash
systemctl --user restart clawdbot-gateway

# Verify the model is recognized
clawdbot models list | grep -i claude
```

**Available Bedrock Claude Model IDs:**

> ⚠️ **IMPORTANT: Cross-Region Inference Profiles Required!**
>
> Newer Claude models (Claude 3.5+, Claude 4+) in `us-east-1` require using the **cross-region inference profile ID** (prefixed with `us.`) instead of the raw model ID.
>
> - ❌ Raw ID: `anthropic.claude-opus-4-5-20251101-v1:0` → Fails with "on-demand throughput isn't supported"
> - ✅ Profile ID: `us.anthropic.claude-opus-4-5-20251101-v1:0` → Works!

| Model | Raw Model ID | Cross-Region Profile ID (use this!) |
|-------|--------------|-------------------------------------|
| **Claude Opus 4.5** | `anthropic.claude-opus-4-5-20251101-v1:0` | `us.anthropic.claude-opus-4-5-20251101-v1:0` |
| **Claude Sonnet 4.5** | `anthropic.claude-sonnet-4-5-20250929-v1:0` | `us.anthropic.claude-sonnet-4-5-20250929-v1:0` |
| **Claude Opus 4.1** | `anthropic.claude-opus-4-1-20250805-v1:0` | `us.anthropic.claude-opus-4-1-20250805-v1:0` |
| **Claude Sonnet 4** | `anthropic.claude-sonnet-4-20250514-v1:0` | `us.anthropic.claude-sonnet-4-20250514-v1:0` |
| Claude 3.7 Sonnet | `anthropic.claude-3-7-sonnet-20250219-v1:0` | `us.anthropic.claude-3-7-sonnet-20250219-v1:0` |
| Claude 3.5 Sonnet v2 | `anthropic.claude-3-5-sonnet-20241022-v2:0` | `us.anthropic.claude-3-5-sonnet-20241022-v2:0` |
| Claude 3.5 Haiku | `anthropic.claude-3-5-haiku-20241022-v1:0` | `us.anthropic.claude-3-5-haiku-20241022-v1:0` |
| Claude 3 Haiku | `anthropic.claude-3-haiku-20240307-v1:0` | Works without prefix ✅ |
| Claude 3 Sonnet | `anthropic.claude-3-sonnet-20240229-v1:0` | Works without prefix ✅ |
| Claude 3 Opus | `anthropic.claude-3-opus-20240229-v1:0` | Works without prefix ✅ |

**Recommended for ClawdBot:**
```bash
# Best balance of capability and cost
us.anthropic.claude-sonnet-4-5-20250929-v1:0

# Most capable (higher cost)
us.anthropic.claude-opus-4-5-20251101-v1:0

# Budget-friendly
us.anthropic.claude-3-5-haiku-20241022-v1:0
```

- [ ] **CHECKPOINT:** ClawdBot configured for AWS Bedrock

#### Step 1.11.5: Test the Configuration

```bash
# Test that ClawdBot can reach Bedrock
clawdbot health

# Start a test conversation (from EC2)
clawdbot chat "Hello, can you confirm you're running via AWS Bedrock?"
```

Then from your **local machine** with the SSH tunnel:
1. Open `http://localhost:18789/?token=YOUR_TOKEN`
2. Start a conversation in the dashboard
3. Verify responses are coming through!

- [ ] **CHECKPOINT:** AWS Bedrock integration working

### Step 1.12: Enable Advanced Memory Features (HIGHLY RECOMMENDED)

> 💡 **Source:** [Memory Features Tip (X Post)](https://x.com/mayloidy/status/1884451877418135613) - Community discovery of these hidden defaults

By default, ClawdBot's two best memory features are **turned OFF**. Enabling them dramatically improves context retention across sessions and compactions.

#### What These Features Do

| Feature | Setting | Problem It Solves |
|---------|---------|-------------------|
| **Memory Flush** | `compaction.memoryFlush.enabled: true` | When context gets too large, ClawdBot "compacts" it (summarizes old messages). Without memory flush, important details get lost. With it enabled, everything important is automatically saved to memory files RIGHT BEFORE compaction. |
| **Session Memory Search** | `memorySearch.experimental.sessionMemory: true` | Normally ClawdBot can only search your `MEMORY.md` file. With session memory search, it can search through **every conversation** you've ever had, even ones it no longer "remembers." |

#### Why Enable These?

- ✅ **Zero performance cost** - These are just configuration flags
- ✅ **Prevents data loss** - Memory flush saves context before compaction wipes it
- ✅ **Better recall** - Session search finds answers from past conversations without you repeating context
- ✅ **Stable features** - Marked experimental but have been reliable

#### Apply the Configuration

**Option A: Via Gateway API** (if gateway is running with tools):

Ask ClawdBot:
```
Enable memory flush before compaction and session memory search in my Clawdbot config.
Set compaction.memoryFlush.enabled to true and set memorySearch.experimental.sessionMemory
to true with sources including both memory and sessions. Apply the config changes.
```

**Option B: Manual Edit** (on EC2):

```bash
# Edit the config file
nano ~/.clawdbot/clawdbot.json
```

Add/update these sections in your `agents.defaults`:

```json
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "sources": ["memory", "sessions"],
        "experimental": {
          "sessionMemory": true
        }
      },
      "compaction": {
        "mode": "safeguard",
        "memoryFlush": {
          "enabled": true
        }
      }
    }
  }
}
```

**Restart the gateway:**
```bash
systemctl --user restart clawdbot-gateway
```

#### Verify Configuration

```bash
# Check the config shows the new settings
cat ~/.clawdbot/clawdbot.json | grep -A5 "memoryFlush"
cat ~/.clawdbot/clawdbot.json | grep -A5 "memorySearch"
```

Expected output should show:
- `"memoryFlush": { "enabled": true }`
- `"sessionMemory": true`
- `"sources": ["memory", "sessions"]`

- [ ] **CHECKPOINT:** Memory flush enabled
- [ ] **CHECKPOINT:** Session memory search enabled

---

#### Alternative: Direct Anthropic API Key

If you prefer to use Anthropic directly (requires separate API key from [console.anthropic.com](https://console.anthropic.com/)):

```bash
# On EC2:
clawdbot configure --section auth --set provider=anthropic
clawdbot configure --section auth --set anthropic.apiKey=sk-ant-api03-xxxxx
systemctl --user restart clawdbot-gateway
```

- [ ] **CHECKPOINT:** AI provider configured (Bedrock OR Anthropic)

---

## Phase 2: Security Lockdown

### ⚠️ CRITICAL: This phase is NON-NEGOTIABLE!

### Step 2.1: Verify Gateway is Bound to Loopback

```bash
# Run security audit
clawdbot security audit --deep

# Check the gateway configuration
clawdbot configure --section gateway

# If bind is NOT "loopback", IMMEDIATELY fix it:
clawdbot configure --section gateway --set bind=loopback
systemctl --user restart clawdbot-gateway
```

- [ ] **CHECKPOINT:** Gateway bind = loopback

### Step 2.2: Verify Port 18789 is NOT Externally Accessible

From your **local machine** (NOT the EC2):
```bash
# This should FAIL/timeout (that's what we want!)
nc -zv <ELASTIC_IP> 18789
# Expected: Connection refused or timeout

# Or test multiple ports:
nc -zv <ELASTIC_IP> 22 80 443 18789 2>&1 | grep -E "(succeeded|refused)"
# Port 22 should succeed, 18789 should NOT appear or show refused
```

From **inside the EC2**:
```bash
# This should SUCCEED
nc -zv 127.0.0.1 18789
# Expected: Connection succeeded
```

- [ ] **CHECKPOINT:** Port 18789 not externally accessible

### Step 2.3: Set Up SSH Tunnel for ClawdBot Access

From your **local machine**:
```bash
# This forwards local port 18789 to the EC2's localhost:18789
ssh -i ~/.ssh/clawdbot-key.pem -L 18789:127.0.0.1:18789 -N ec2-user@<ELASTIC_IP>
```

Now open in your browser:
```
http://localhost:18789/?token=YOUR_GATEWAY_TOKEN
```

**Get your token** (on EC2):
```bash
cat ~/.clawdbot/clawdbot.json | grep -A2 '"auth"'
# Or check onboarding output for the tokenized URL
```

- [ ] **CHECKPOINT:** SSH tunnel working

### Step 2.4: Restrict SSH to Your IP Only

Get your current public IP:
```bash
curl -s ifconfig.me
```

In **CloudShell**:
```bash
# Load saved environment
source clawdbot-instance.env

YOUR_IP="<YOUR_PUBLIC_IP_HERE>"

# Remove the open SSH rule
aws ec2 revoke-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Add rule for your IP only
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr ${YOUR_IP}/32

echo "✅ SSH now restricted to $YOUR_IP"
```

**⚠️ Remember:** If your IP changes (different WiFi, VPN, etc.), you'll need to update this rule!

- [ ] **CHECKPOINT:** SSH restricted to your IP

### Step 2.5: Create SSH Config for Easy Access

On your **local machine**, add to `~/.ssh/config`:
```
Host clawdbot
    HostName <ELASTIC_IP>
    User ec2-user
    IdentityFile ~/.ssh/clawdbot-key.pem
    LocalForward 18789 127.0.0.1:18789
```

Now you can connect with just:
```bash
ssh clawdbot
# This also sets up the tunnel automatically!
```

- [ ] **CHECKPOINT:** SSH config created

### Step 2.6: Security Scan with Shodan/Nmap

**Option 1: Shodan (online)**
1. Go to [shodan.io](https://www.shodan.io/)
2. Search for your Elastic IP
3. Verify only ports 22, 80, 443 are visible
4. Port 18789 should NOT be listed!

**Option 2: Nmap (from your Mac)**
```bash
# Install if needed: brew install nmap
nmap -Pn <ELASTIC_IP>
```

**Expected results:**
- ✅ Port 22/tcp open ssh
- ✅ Port 80/tcp open http (or filtered if no service)
- ✅ Port 443/tcp open https (or filtered if no service)
- ❌ Port 18789 should NOT appear!

- [ ] **CHECKPOINT:** Security scan passed

### Step 2.7: Advanced Security Hardening (Daniel Miessler Checklist)

Based on the [CLAWDBOT Security Hardening](https://x.com/DanielMiessler/status/2015865548714975475) checklist, implement these additional security measures:

#### 2.7.1: Verify/Set Gateway Auth Token

```bash
# On EC2: View current config
cat ~/.clawdbot/clawdbot.json

# Generate a strong random token
TOKEN=$(openssl rand -hex 32)
echo "Your new token: $TOKEN"

# Backup config first
cp ~/.clawdbot/clawdbot.json ~/.clawdbot/clawdbot.json.bak

# Edit config to update/add the token in the gateway.auth section
nano ~/.clawdbot/clawdbot.json
# Find "auth" section under gateway and set: "token": "YOUR_TOKEN_HERE"

# Or use jq if installed:
# jq --arg token "$TOKEN" '.gateway.auth.token = $token' ~/.clawdbot/clawdbot.json > tmp.json && mv tmp.json ~/.clawdbot/clawdbot.json

# Restart gateway after changes
systemctl --user restart clawdbot-gateway
```

- [ ] **CHECKPOINT:** Gateway auth token is cryptographic random

#### 2.7.2: Set DM Policy to Allowlist

```bash
# On EC2: Check current DM policy
clawdbot configure --section permissions

# Set to allowlist mode with explicit users
clawdbot configure --section permissions --set dm_policy=allowlist
clawdbot configure --section permissions --set allowed_users='["your-user-id"]'

# Verify
clawdbot configure --section permissions
```

- [ ] **CHECKPOINT:** DM policy set to allowlist

#### 2.7.3: Enable Sandbox Mode

```bash
# On EC2: Enable sandbox for all operations
clawdbot configure --section security --set sandbox=all

# If using Docker, add network isolation:
# clawdbot configure --section security --set docker.network=none

# Verify sandbox status
clawdbot security audit --deep
```

- [ ] **CHECKPOINT:** Sandbox enabled

#### 2.7.4: Secure Credentials (No Plaintext!)

```bash
# On EC2: Check file permissions on config files
ls -la ~/.clawdbot/

# Set restrictive permissions
chmod 600 ~/.clawdbot/clawdbot.json
chmod 600 ~/.clawdbot/*.json

# Move sensitive credentials to environment variables
# Edit your shell profile:
cat >> ~/.bashrc << 'EOF'
# ClawdBot credentials (never commit to git!)
export ANTHROPIC_API_KEY="your-key-here"
EOF

# Update clawdbot to use env var instead of file
clawdbot configure --section auth --set anthropic.apiKey='${ANTHROPIC_API_KEY}'

# Verify no plaintext credentials
grep -r "sk-" ~/.clawdbot/ 2>/dev/null && echo "⚠️ Plaintext keys found!" || echo "✅ No plaintext keys"
```

- [ ] **CHECKPOINT:** Credentials secured (env vars + chmod 600)

#### 2.7.5: Block Dangerous Commands

```bash
# On EC2: Add command blocklist
clawdbot configure --section security --set blocked_commands='[
  "rm -rf /",
  "rm -rf ~",
  "rm -rf /*",
  "curl * | bash",
  "curl * | sh",
  "wget * | bash",
  "git push --force",
  "git push -f",
  "chmod -R 777",
  ":(){ :|:& };:"
]'

# Verify blocklist
clawdbot configure --section security
```

- [ ] **CHECKPOINT:** Dangerous commands blocked

#### 2.7.6: Restrict MCP Tools to Minimum Needed

```bash
# On EC2: Review currently enabled tools/skills
clawdbot configure --section skills

# Disable any tools you don't need:
# clawdbot configure --section skills --set browser=false
# clawdbot configure --section skills --set filesystem=restricted

# For MCP servers, only enable what's necessary
clawdbot mcp list
# Disable unused MCP servers
```

- [ ] **CHECKPOINT:** MCP tools restricted to minimum

#### 2.7.7: Enable Comprehensive Audit Logging

```bash
# On EC2: Enable detailed logging
clawdbot configure --section logging --set level=verbose
clawdbot configure --section logging --set audit=true
clawdbot configure --section logging --set log_file=~/.clawdbot/audit.log

# Set up log rotation
sudo tee /etc/logrotate.d/clawdbot << 'EOF'
/home/ec2-user/.clawdbot/audit.log {
    daily
    rotate 30
    compress
    missingok
    notifempty
}
EOF

# Verify logging is working
tail -f ~/.clawdbot/audit.log
```

- [ ] **CHECKPOINT:** Audit logging enabled

#### 2.7.8: Network Isolation (Docker Option)

For maximum isolation, run ClawdBot in Docker with network restrictions:

```bash
# On EC2: Create isolated Docker network (optional advanced setup)
# sudo dnf install -y docker
# sudo systemctl start docker
# sudo systemctl enable docker

# Run with no external network access:
# docker run --network=none -v ~/.clawdbot:/root/.clawdbot clawdbot/gateway
```

**Note:** This is optional for advanced users. The loopback binding + SSH tunnel already provides strong isolation.

- [ ] **CHECKPOINT:** Network isolation configured (or skipped if using SSH tunnel)

#### Security Hardening Summary

| Fix | Command to Verify | Expected Result |
|-----|-------------------|-----------------|
| Auth token | `clawdbot configure --section gateway` | Token should be 32+ hex chars |
| DM policy | `clawdbot configure --section permissions` | `dm_policy: allowlist` |
| Sandbox | `clawdbot security audit --deep` | Sandbox: enabled |
| Credentials | `ls -la ~/.clawdbot/*.json` | `-rw-------` (600 perms) |
| Commands | `clawdbot configure --section security` | Blocklist populated |
| MCP tools | `clawdbot mcp list` | Only needed tools enabled |
| Logging | `tail ~/.clawdbot/audit.log` | Logs appearing |

---

# PART 2: DIGITAL TWIN / VA SETUP

> **Goal:** Give Clawdbot its own identity to act autonomously — email, GitHub, social media.
> **Who needs this:** Those building a personal AI assistant / digital twin.
> **Prerequisites:** Complete Part 1 first.

---

## 🔐 Channel Security Comparison

Before setting up external channels, understand the security tradeoffs:

| Channel | Security | Why |
|---------|----------|-----|
| **SSH tunnel + Web chat** | ✅ Excellent | Only you have the PEM key |
| **Tailscale VPN** | ✅ Excellent | Encrypted mesh, device-based auth |
| **X/Twitter DMs** | ✅ Good | Private DMs, you control the account |
| **Email** | ✅ Good | Private inbox, you own the domain |
| **Telegram Bot** | ⚠️ Moderate | Can allowlist your user ID only |
| **WhatsApp** | ⚠️ Risky | Borrows YOUR identity — AI replies as you |
| **Discord (public server)** | ❌ Risky | Anyone in server can interact |

**Recommendation:** Start with SSH tunnel (Part 1). Add email + X for external communication. Avoid WhatsApp unless you use a dedicated number.

---

## Phase 3: Clawdbot Identity (Email + GitHub + X)

For Clawdbot to autonomously manage repos, post to social media, receive notifications, and act as your digital twin, it needs its own identity.

### The Identity Chain

```
Email (ai@yourdomain.com)          ← Foundation (required first)
    │
    ├── GitHub account             ← Code, repos, commits
    ├── X/Twitter account          ← Social presence, posts
    └── Other services             ← As needed (sign up with email)
```

### Why Give Clawdbot Its Own Identity?

| Capability | Requires |
|------------|----------|
| Push commits to repos | GitHub account |
| Post to X/Twitter | X account |
| Receive GitHub notifications | Email address |
| Sign up for any service | Email address |
| Act independently | Distinct identity |

### Step 3.1: Set Up AWS WorkMail (Recommended)

**Why WorkMail over SES?**
- SES = API only, no inbox, dumps emails to S3
- WorkMail = Full mailbox with IMAP, like Google Workspace

**WorkMail gives you:**
- ✅ Real inbox (IMAP access for Clawdbot via CLI)
- ✅ Web UI (you can log in and review anytime)
- ✅ Folders, sent mail, search
- ✅ Uses your AWS credits
- ✅ Calendar & contacts included

**Cost:** $4/user/month (~$48/year)

**Setup Steps:**

1. **Go to AWS WorkMail Console**: [console.aws.amazon.com/workmail](https://console.aws.amazon.com/workmail/)

2. **Create an organization:**
   ```
   WorkMail Console → Create organization → Quick setup
   Name: yourdomain-mail (or similar)
   ```

3. **Add your domain:**
   ```
   Organization → Domains → Add domain → yourdomain.com
   ```

4. **Add DNS records** (WorkMail will provide these):
   - MX record (for receiving mail)
   - SPF record (TXT)
   - DKIM records (CNAME)
   - Autodiscover (CNAME) - optional

5. **Create the AI mailbox:**
   ```
   Organization → Users → Create user
   Username: ai
   Display name: Your AI Assistant
   Email: ai@yourdomain.com
   ```

6. **Note the IMAP/SMTP servers:**
   ```
   IMAP: imap.mail.us-east-1.awsapps.com (port 993, SSL)
   SMTP: smtp.mail.us-east-1.awsapps.com (port 465, SSL)
   ```

**Web UI access:** `https://mail.us-east-1.awsapps.com/mail`

- [ ] **CHECKPOINT:** WorkMail organization created
- [ ] **CHECKPOINT:** Domain verified
- [ ] **CHECKPOINT:** ai@yourdomain.com mailbox created

### Step 3.2: Install Himalaya CLI (Agent-Native Email)

**Why Himalaya?**
- CLI-native (not built for humans clicking buttons)
- JSON output (easy for AI to parse)
- Rust-based, fast, actively maintained
- GitHub: [pimalaya/himalaya](https://github.com/pimalaya/himalaya)

```bash
# On EC2:
ssh clawdbot

# Install via cargo (if Rust installed)
cargo install himalaya

# Or download pre-built binary
curl -LO https://github.com/pimalaya/himalaya/releases/latest/download/himalaya-linux-x86_64.tar.gz
tar -xzf himalaya-linux-x86_64.tar.gz
sudo mv himalaya /usr/local/bin/
himalaya --version
```

**Configure Himalaya:**

```bash
mkdir -p ~/.config/himalaya

cat > ~/.config/himalaya/config.toml << 'EOF'
[ai@yourdomain.com]
email = "ai@yourdomain.com"
display-name = "Your AI Assistant"

backend = "imap"
imap-host = "imap.mail.us-east-1.awsapps.com"
imap-port = 993
imap-ssl = true

sender = "smtp"
smtp-host = "smtp.mail.us-east-1.awsapps.com"
smtp-port = 465
smtp-ssl = true

# Password will be prompted on first use, or use keyring
EOF
```

**Test Himalaya:**

```bash
# List inbox (JSON output for agent parsing)
himalaya --output json list

# Read specific email
himalaya --output json read 1

# Send an email
himalaya send --to someone@example.com --subject "Test" --body "Hello from Clawdbot!"

# Reply to an email
himalaya reply 1 --body "Thanks for your email!"
```

- [ ] **CHECKPOINT:** Himalaya installed and configured
- [ ] **CHECKPOINT:** Can list/read/send emails via CLI

### Oversight Model

```
┌─────────────────────────────────────────────────────────┐
│                 ai@yourdomain.com                        │
│                 (Same mailbox)                           │
├────────────────────────┬────────────────────────────────┤
│                        │                                 │
│  YOU (Human)           │  CLAWDBOT (AI)                 │
│  WorkMail Web UI       │  Himalaya CLI                  │
│  (browser login)       │  (IMAP connection)             │
│                        │                                 │
│  • Review all emails   │  • Read inbox                  │
│  • See sent messages   │  • Send replies                │
│  • Check what AI did   │  • Compose new emails          │
│  • Override if needed  │  • Manage folders              │
│                        │                                 │
└────────────────────────┴────────────────────────────────┘
```

**Full visibility** — You can log into the WorkMail web UI anytime to see every email Clawdbot sends/receives.

### Step 3.3: Create GitHub Account for Clawdbot

1. **Go to** [github.com/signup](https://github.com/signup)
2. **Use the email** you created (e.g., `ai@yourdomain.com`)
3. **Username suggestion:** `yourdomain-ai` or `yourdomain-bot`
4. **Verify email** via the link sent to WorkMail
5. **Enable 2FA** for security

- [ ] **CHECKPOINT:** GitHub account created

### Step 3.4: Generate GitHub Personal Access Token (PAT)

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **"Generate new token (classic)"**
3. Set permissions:
   - ✅ `repo` (full control)
   - ✅ `workflow` (if using Actions)
   - ✅ `read:org` (if collaborating on org repos)
4. **Copy the token** (starts with `ghp_`)

- [ ] **CHECKPOINT:** GitHub PAT generated

### Step 3.5: Configure Clawdbot with GitHub Access

SSH into your EC2:

```bash
ssh clawdbot
```

**Add GitHub credentials to systemd service:**

```bash
# Create/update the systemd drop-in file
cat >> ~/.config/systemd/user/clawdbot-gateway.service.d/aws.conf << 'EOF'
Environment="GITHUB_TOKEN=ghp_YOUR_TOKEN_HERE"
EOF

# Reload and restart
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

**Configure git identity on EC2:**

```bash
git config --global user.name "Your AI Assistant"
git config --global user.email "ai@yourdomain.com"
```

- [ ] **CHECKPOINT:** Clawdbot has GitHub access

### Step 3.6: Add Clawdbot as Collaborator to Repos

For each repo you want Clawdbot to manage:

1. Go to repo → Settings → Collaborators
2. Add the Clawdbot GitHub username
3. Grant **Write** or **Admin** access as needed

Now Clawdbot can:
- ✅ Clone private repos
- ✅ Push commits
- ✅ Create branches and PRs
- ✅ Receive notifications via email

- [ ] **CHECKPOINT:** Clawdbot added as collaborator

### Step 3.7: Create X/Twitter Account for Clawdbot

**X API Pricing (as of 2024):**

| Tier | Cost | Write Access |
|------|------|--------------|
| Free | $0 | ❌ Read-only, very limited |
| Basic | $100/month | ✅ 50k tweets/month |
| Pro | $5,000/month | ✅ Full access |

**Recommended approach: Browser Automation (Free)**

Instead of paying $100/month for API access, Clawdbot can use browser automation to post:

```
Clawdbot has browser tools
        │
        ▼
Login to X (one-time, save session)
        │
        ▼
Navigate to compose
        │
        ▼
Type and post
```

**Setup Steps:**

1. **Create X account:**
   - Go to [x.com/signup](https://x.com/signup)
   - Use `ai@yourdomain.com` email
   - Username suggestion: `yourdomain_ai` or `yourname_ai`
   - Verify email

2. **Login via Clawdbot's browser:**
   ```bash
   # Clawdbot can open browser, login, and save session
   # Then post via browser automation
   ```

3. **Posting workflow:**
   ```
   You: "Post to X: Just shipped a new feature! 🚀"
   
   Clawdbot:
   1. Opens X in browser (using saved session)
   2. Navigates to compose
   3. Types the tweet
   4. Clicks Post
   5. Confirms success
   ```

**Note:** This is more fragile than API access but saves $100/month. If you need reliable high-volume posting, consider the Basic API tier.

- [ ] **CHECKPOINT:** X account created
- [ ] **CHECKPOINT:** Can post via browser automation

### Clawdbot Digital Identity Summary

| Service | Handle | Purpose | Status |
|---------|--------|---------|--------|
| Email | ai@yourdomain.com | Foundation for all services | 🔜 |
| GitHub | @yourdomain-ai | Code, repos, commits | 🔜 |
| X/Twitter | @yourdomain_ai | Social posts | 🔜 |
| Website | yourdomain.com | Public presence (you build, Clawdbot maintains) | 🔜 |

---

## Phase 4: Public Website (Hosted Separately)

> ⚠️ **Do NOT host your public website on this EC2!**
>
> For security, keep this Clawdbot server private. Host your public website on a separate platform.

### Recommended Hosting Options

| Platform | Best For | Cost |
|----------|----------|------|
| **Vercel** | Next.js, React | Free tier |
| **Netlify** | Static sites, JAMstack | Free tier |
| **Cloudflare Pages** | Static + Workers | Free tier |
| **S3 + CloudFront** | AWS-native static | ~$1/month |

### How Clawdbot Updates Your Website

```
┌─────────────────────┐         ┌─────────────────────┐
│  Clawdbot (EC2)     │         │  Vercel/Netlify     │
│                     │         │                     │
│  1. You instruct    │         │  3. Auto-deploys    │
│     Clawdbot        │ ──git──▶│     from main       │
│                     │  push   │                     │
│  2. Clawdbot edits  │         │  4. Site updated!   │
│     files & commits │         │                     │
└─────────────────────┘         └─────────────────────┘
```

### Step 4.1: Create Website Repo

On GitHub (using Clawdbot's account or yours):

```bash
# Create a new repo for your website
# e.g., github.com/yourusername/yourdomain-website
```

### Step 4.2: Deploy to Vercel (Recommended)

1. Go to [vercel.com](https://vercel.com)
2. Sign up / log in with GitHub
3. Import your website repo
4. Configure:
   - Framework: Next.js (or your choice)
   - Root directory: `/` (or your app folder)
5. Deploy!

**Add custom domain:**
```
Vercel Dashboard → Project → Settings → Domains → Add yourdomain.com
```

**Configure DNS** (in Route 53 or your registrar):
```
Type: CNAME
Name: www (or @)
Value: cname.vercel-dns.com
```

- [ ] **CHECKPOINT:** Website deployed to Vercel

### Step 4.3: Public Chatbot (Optional)

For a public chatbot on your website, add a simple API route:

```typescript
// pages/api/chat.ts (Next.js example)
import Anthropic from "@anthropic-ai/sdk";

export default async function handler(req, res) {
  const { message } = req.body;
  
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY // Vercel env var
  });
  
  const response = await client.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 1024,
    system: "You are a helpful assistant for [your website]...",
    messages: [{ role: "user", content: message }]
  });
  
  res.json({ reply: response.content[0].text });
}
```

**This public chatbot:**
- ❌ Does NOT connect to your private Clawdbot
- ✅ Uses direct Anthropic/Bedrock API calls
- ✅ Is stateless (no memory, no tools)
- ✅ Safe for public exposure

- [ ] **CHECKPOINT:** Public chatbot deployed (optional)

---

## Phase 5: Crabwalk Integration

### What is Crabwalk?
Open-source companion monitor for ClawdBot (https://github.com/luccast/crabwalk)

Features:
- Live node graph of sessions & action chains
- WhatsApp, Telegram, Discord, Slack integration
- See thinking states, tool calls, responses in real-time
- Filter by platform, search by recipient

### Planned Steps:
- [ ] Clone crabwalk repo
- [ ] Configure to connect to ClawdBot gateway
- [ ] Run as a monitoring dashboard

---

## Troubleshooting

### Can't SSH into instance

**Symptoms:** Connection hangs or times out

**Solutions:**
1. **Wait 2 minutes** - Instance may still be booting after start/reboot
2. **Check instance state** (in CloudShell):
   ```bash
   aws ec2 describe-instances --instance-ids $INSTANCE_ID \
       --query 'Reservations[0].Instances[0].State.Name'
   ```
3. **Check security group rules**:
   ```bash
   aws ec2 describe-security-groups --group-ids $SG_ID \
       --query 'SecurityGroups[0].IpPermissions'
   ```
   If empty or missing port 22, re-add the SSH rule
4. **Your IP may have changed** - Update the SSH security group rule
5. **Check PEM key permissions**: `ls -la ~/.ssh/clawdbot-key.pem` should show `-r--------`

### "Permission denied (publickey)" when SSH

**Cause:** Invalid PEM key format (often from TextEdit adding formatting)

**Solution:**
```bash
# Delete and recreate the key
rm ~/.ssh/clawdbot-key.pem
# Use cat method to create it (see Step 1.5 Option B)
```

### "No space left on device" during npm install

**Cause:** Default 8GB EBS volume is too small

**Solution 1: Resize existing volume** (in CloudShell):
```bash
# Get volume ID
VOLUME_ID=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId' \
    --output text)

# Resize to 20GB
aws ec2 modify-volume --volume-id $VOLUME_ID --size 20
```

Then on EC2:
```bash
sudo growpart /dev/xvda 1
sudo xfs_growfs /
df -h  # Verify new size
```

**Solution 2: Clean up first**:
```bash
npm cache clean --force
rm -rf ~/.npm/_cacache
```

### ClawdBot gateway not starting

```bash
# Check service status
systemctl --user status clawdbot-gateway

# View logs
journalctl --user -u clawdbot-gateway -f

# Try manual start for debugging
clawdbot gateway --port 18789 --verbose
```

### Port 18789 accessible from outside (SECURITY ISSUE!)

**IMMEDIATELY run on EC2:**
```bash
clawdbot configure --section gateway --set bind=loopback
systemctl --user restart clawdbot-gateway
```

**Verify it's fixed:**
```bash
# From your Mac (should fail/timeout):
nc -zv <ELASTIC_IP> 18789
```

### IP changed after reboot

If you didn't set up an Elastic IP:
```bash
# In CloudShell, get new IP:
aws ec2 describe-instances --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].PublicIpAddress' --output text
```

**Fix:** Allocate an Elastic IP (see Step 1.4)

### AWS Bedrock "Missing auth" or "Unknown model" errors

**Symptoms:**
- `clawdbot models` shows "Missing auth" for amazon-bedrock
- Error: `Unknown model: amazon-bedrock/...`
- Gateway crashes on startup with config validation errors

**Root Causes & Solutions:**

1. **Systemd service doesn't see AWS credentials**
   
   The Clawdbot gateway runs as a systemd service, which is isolated from your shell environment. Even if `aws sts get-caller-identity` works in SSH, the service won't see those credentials.
   
   **Fix:** Create a systemd drop-in file (see Step 1.11.3 Option B):
   ```bash
   mkdir -p ~/.config/systemd/user/clawdbot-gateway.service.d
   cat > ~/.config/systemd/user/clawdbot-gateway.service.d/aws.conf << 'EOF'
   [Service]
   Environment="AWS_ACCESS_KEY_ID=YOUR_KEY"
   Environment="AWS_SECRET_ACCESS_KEY=YOUR_SECRET"
   Environment="AWS_REGION=us-east-1"
   EOF
   systemctl --user daemon-reload
   systemctl --user restart clawdbot-gateway
   ```

2. **Using raw model ID instead of cross-region inference profile**
   
   Newer Claude models require the `us.` prefix for cross-region inference.
   
   - ❌ `amazon-bedrock/anthropic.claude-opus-4-5-20251101-v1:0`
   - ✅ `amazon-bedrock/us.anthropic.claude-opus-4-5-20251101-v1:0`

3. **Invalid auth profile format in config**
   
   Clawdbot does NOT support embedded AWS credentials in `clawdbot.json`. This format is INVALID:
   ```json
   "amazon-bedrock:default": {
     "provider": "amazon-bedrock",
     "mode": "keys",
     "accessKeyId": "...",  // ❌ NOT SUPPORTED
     "secretAccessKey": "..."
   }
   ```
   
   **Fix:** Remove invalid auth profiles and use systemd environment variables instead.

4. **AWS Marketplace subscription not activated**
   
   Error: `AccessDeniedException ... aws-marketplace:Subscribe`
   
   **Fix:** Log into AWS Console as admin, go to Bedrock Playground, select the model, and send a test message. This triggers the one-time marketplace subscription acceptance.

**Verify Bedrock is working:**
```bash
# Test AWS SDK directly (should work if credentials are correct)
python3 -c "
import boto3
client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.invoke_model(
    modelId='us.anthropic.claude-3-haiku-20240307-v1:0',
    body='{\"anthropic_version\":\"bedrock-2023-05-31\",\"max_tokens\":10,\"messages\":[{\"role\":\"user\",\"content\":\"Hi\"}]}'
)
print('SUCCESS!' if response else 'FAILED')
"

# Check Clawdbot sees the credentials
clawdbot models
# Should NOT show "Missing auth" for amazon-bedrock
```

---

## Quick Reference Commands

### AWS CloudShell
```bash
# Instance status
aws ec2 describe-instances --instance-ids $INSTANCE_ID \
    --query 'Reservations[0].Instances[0].[State.Name,PublicIpAddress]'

# Security group rules
aws ec2 describe-security-groups --group-ids $SG_ID \
    --query 'SecurityGroups[0].IpPermissions'

# Reboot instance
aws ec2 reboot-instances --instance-ids $INSTANCE_ID
```

### ClawdBot (on EC2)
```bash
clawdbot status              # Check overall status
clawdbot health              # Health check
clawdbot gateway status      # Gateway status
clawdbot security audit --deep  # Security audit
clawdbot configure --section gateway  # View gateway config

# Service management
systemctl --user start clawdbot-gateway
systemctl --user stop clawdbot-gateway
systemctl --user restart clawdbot-gateway
systemctl --user status clawdbot-gateway
```

### SSH Tunnel (from local machine)
```bash
# Simple tunnel
ssh -i ~/.ssh/clawdbot-key.pem -L 18789:127.0.0.1:18789 -N ec2-user@<ELASTIC_IP>

# Or if you set up ~/.ssh/config:
ssh clawdbot
```

### Access ClawdBot Dashboard (after tunnel)
```
http://localhost:18789/?token=YOUR_TOKEN
```

---

## Notes & Reminders

- ❌ **Never** expose port 18789 to the public internet
- ✅ **Always** use SSH tunnel to access ClawdBot
- 🔄 Update your IP in security group if it changes
- 💾 Back up `~/.clawdbot/` directory regularly
- 🔄 Keep Node.js and ClawdBot updated
- 💰 Elastic IP is free when attached to running instance

---

## Progress Tracking

### Phase 1: AWS VPS Setup ✅ COMPLETE
| Step | Status | Date |
|------|--------|------|
| 1.1 Create key pair | ✅ Done | 2026-01-27 |
| 1.2 Create security group | ✅ Done | 2026-01-27 |
| 1.3 Launch EC2 instance | ✅ Done | 2026-01-27 |
| 1.4 Allocate Elastic IP | ✅ Done | 2026-01-27 |
| 1.5 Save PEM key locally | ✅ Done | 2026-01-27 |
| 1.6 SSH into instance | ✅ Done | 2026-01-27 |
| 1.7 Install Node.js 22+ | ✅ Done | 2026-01-27 |
| 1.8 Install ClawdBot | ✅ Done | 2026-01-27 |
| 1.9 Run onboarding | ✅ Done | 2026-01-27 |
| 1.10 Start gateway | ✅ Done | 2026-01-27 |
| 1.11 Configure AWS Bedrock | ✅ Done | 2026-01-28 |
| 1.12 Enable memory features | ✅ Done | 2026-01-28 |

**Notes:**
- Upgraded to t2.small (2GB RAM) due to OOM errors on t2.micro
- Added 2GB swap for stability
- Elastic IP allocated and associated ✅
- Dashboard accessible via SSH tunnel ✅

### Phase 2: Security Lockdown ✅ COMPLETE
| Step | Status | Date |
|------|--------|------|
| 2.1 Verify loopback binding | ✅ Done (via QuickStart) | 2026-01-27 |
| 2.2 Verify port blocked | ✅ Done (nc timeout = secure!) | 2026-01-27 |
| 2.3 Set up SSH tunnel | ✅ Done | 2026-01-27 |
| 2.4 Restrict SSH to IP | ⏭️ Skipped (optional) | |
| 2.5 Create SSH config | ✅ Done | 2026-01-27 |
| 2.6 Security scan | ✅ Done (nc verified) | 2026-01-27 |

**Security Summary:**
- Port 18789: NOT accessible from internet ✅
- Port 22: Open (SSH access) ✅
- Gateway bound to localhost only ✅
- SSH config: `ssh clawdbot` auto-tunnels ✅

### Phase 2.7: Advanced Security Hardening (Daniel Miessler Checklist) ✅ CORE COMPLETE
| # | Vulnerability | Fix | Status |
|---|--------------|-----|--------|
| 1 | Gateway exposed on 0.0.0.0:18789 | Bind to loopback | ✅ Done (`bind: loopback`) |
| 2 | DM policy allows all users | Set dm_policy to allowlist | ⏭️ N/A (no channels configured) |
| 3 | Sandbox disabled by default | Enable sandbox=all | ⏭️ Advanced (Docker optional) |
| 4 | Credentials in plaintext | Use chmod 600 permissions | ✅ Done (`-rw-------`) |
| 5 | Prompt injection via web content | Wrap untrusted content | 📝 N/A (no web content yet) |
| 6 | Dangerous commands unblocked | Block rm -rf, curl pipes | ⏭️ Advanced feature |
| 7 | No network isolation | SSH tunnel provides isolation | ✅ Done (port not exposed) |
| 8 | Elevated tool access granted | Restrict MCP tools | ⏭️ Configure as needed |
| 9 | No audit logging enabled | Enable session logging | ⏭️ Optional |
| 10 | Weak/default pairing codes | Cryptographic random token | ✅ Done (48-char hex) |

**Reference:** [Daniel Miessler's CLAWDBOT Security Hardening](https://x.com/DanielMiessler/status/2015865548714975475)

---

*Document updated as we progressed through setup.*

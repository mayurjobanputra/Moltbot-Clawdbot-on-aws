# ClawdBot AWS VPS Setup Guide

**Author:** AI-assisted setup documentation  
**Last Updated:** 2026-01-27
**Status:** Phase 2 - Security Lockdown (In Progress)

---

## Table of Contents
1. [Overview](#overview)
2. [Phase 0: AWS Account Setup](#phase-0-aws-account-setup-before-you-begin)
3. [Phase 1: AWS VPS Setup](#phase-1-aws-vps-setup)
4. [Phase 2: Security Lockdown](#phase-2-security-lockdown)
5. [Phase 3: VR Website Setup](#phase-3-vr-website-setup)
6. [Phase 4: Domain Configuration (mayur.ai)](#phase-4-domain-configuration)
7. [Phase 5: Crabwalk Integration](#phase-5-crabwalk-integration)
8. [Troubleshooting](#troubleshooting)
9. [Progress Tracking](#progress-tracking)

---

## Overview

### Goals
- Run ClawdBot as a personal AI assistant on an AWS VPS
- Secure the instance so only YOU can access it
- Serve a public website on ports 80/443 via your domain (mayur.ai)
- Keep ClawdBot gateway (port 18789) private and locked down

### Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    AWS EC2 Instance                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌──────────────────────────────────┐│
│  │   ClawdBot      │    │    Your Website (VR Avatar)     ││
│  │   Gateway       │    │    Nginx/Node.js                ││
│  │   Port: 18789   │    │    Ports: 80, 443               ││
│  │   (LOCALHOST    │    │    (PUBLIC ACCESS)              ││
│  │    ONLY!)       │    │                                 ││
│  └─────────────────┘    └──────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                    │                     │
                    │ SSH Tunnel          │ HTTPS
                    │ (Your PEM key)      │ (Public)
                    ▼                     ▼
            ┌───────────────┐     ┌────────────────┐
            │ YOUR LAPTOP   │     │   THE WORLD    │
            │ (Private      │     │   (mayur.ai)   │
            │  Access)      │     │                │
            └───────────────┘     └────────────────┘
```

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

### Step 1.11: Add Anthropic API Key

```bash
# Configure auth
clawdbot configure --section auth

# Or set directly:
# clawdbot configure --section auth --set anthropic.apiKey=YOUR_API_KEY
```

- [ ] **CHECKPOINT:** API key configured

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

## Phase 3: VR Website Setup

*Details to be added once Phase 1 & 2 are complete*

### Planned Steps:
- [ ] Install Nginx
- [ ] Set up Node.js app for VR avatar
- [ ] Configure reverse proxy
- [ ] Test on port 80

---

## Phase 4: Domain Configuration

*Details to be added once Phase 3 is complete*

### Planned Steps:
- [ ] Configure Route 53 for mayur.ai (point to Elastic IP)
- [ ] Set up SSL with Let's Encrypt / certbot
- [ ] Configure Nginx for HTTPS
- [ ] Test https://mayur.ai

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
| 1.11 Add API key | ⏳ Pending (can add later) | |

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

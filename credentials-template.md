# Credentials Template

⚠️ **This is a PUBLIC repository!**

This file is a **TEMPLATE** showing what credentials you'll need. 
- **DO NOT** put real values here
- Copy this to `credentials-local.md` for your actual values (gitignored)

---

## AWS Credentials

| Item | Value | Notes |
|------|-------|-------|
| AWS Account ID | `YOUR_ACCOUNT_ID` | 12-digit number |
| AWS Region | `us-east-1` | Or your preferred region |
| EC2 Instance ID | `i-xxxxxxxxxxxxxxxxx` | Starts with i- |
| EC2 Public IP | `x.x.x.x` | Your instance public IP |
| Security Group ID | `sg-xxxxxxxxxxxxxxxxx` | Starts with sg- |
| Key Pair Name | `clawdbot-key` | Name only, not the file |

---

## SSH Access

| Item | Value | Notes |
|------|-------|-------|
| PEM Key Location | `~/.ssh/clawdbot-key.pem` | Local path |
| SSH Command | `ssh -i ~/.ssh/clawdbot-key.pem ec2-user@<IP>` | Replace IP |

---

## ClawdBot

| Item | Value | Notes |
|------|-------|-------|
| Gateway Port | `18789` | Default port |
| Gateway Token | `YOUR_TOKEN_HERE` | 64-char hex string |
| Dashboard URL (via tunnel) | `http://127.0.0.1:18789/` | Local only |

---

## API Keys

| Service | Key | Notes |
|---------|-----|-------|
| Anthropic API | `sk-ant-...` | For Claude models |
| OpenAI API (optional) | `sk-...` | If using GPT models |
| Brave Search API (optional) | `BSA...` | For web search |

---

## Domain (Phase 4)

| Item | Value | Notes |
|------|-------|-------|
| Domain | `mayur.ai` | Your domain |
| Elastic IP | `x.x.x.x` | Static IP for domain |
| SSL Cert Location | `/etc/letsencrypt/live/mayur.ai/` | After certbot |

---

## Security Notes

1. **Gateway Token**: Generate with `openssl rand -hex 32`
2. **SSH Key**: Never share your `.pem` file
3. **IP Whitelist**: Your current IP for SSH access
4. **API Keys**: Rotate regularly

---

## Quick Setup Commands

```bash
# Generate a strong gateway token
openssl rand -hex 32

# Get your current public IP
curl -s ifconfig.me

# Test SSH tunnel
ssh -i ~/.ssh/clawdbot-key.pem -L 18789:localhost:18789 -N ec2-user@<IP>
```

---

*Create `credentials-local.md` with your actual values - it's gitignored!*

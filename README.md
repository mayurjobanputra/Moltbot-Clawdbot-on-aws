# 🦞 Clawdbot on AWS - Complete Setup Guide

Run [Clawdbot](https://docs.clawd.bot/) (AI agent platform) on an AWS EC2 instance with **Claude, Gemini, GPT, or any LLM** - secured and production-ready.

[![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![Clawdbot](https://img.shields.io/badge/Clawdbot-2026.x-blue)](https://docs.clawd.bot/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 🤖 Let AI Help You Set This Up!

**This guide is designed to be AI-readable.** You can:

1. **Point any LLM at this guide** - Copy the contents of [`CLAWDBOT_AWS_SETUP.md`](CLAWDBOT_AWS_SETUP.md) into ChatGPT, Claude, Gemini, Grok, or any AI assistant and ask it to help you step-by-step.

2. **Use VS Code + Roo Code** - Open this repo in VS Code with [Roo Code](https://roo.ai/) extension. It can read the guide and **execute terminal commands for you** automatically.

3. **Use Claude with Computer Use** - Claude can SSH into your EC2 and configure everything hands-free.

The guide is written to be both human-readable AND machine-parseable with clear checkpoints and commands.

---

## ✨ What You Get

- 🦞 **Clawdbot Gateway** running 24/7 on your own VPS
- 🔒 **Secure by default** - Localhost-only binding, SSH tunnel access
- 💰 **Use your AWS credits** - Works with $1,000 AWS Activate program
- 🧠 **Any LLM backend** - Claude (Bedrock), Gemini, GPT, Anthropic API, and more
- 🧠 **Enhanced memory** - Memory flush + session search (don't lose context!)
- 💬 **Built-in web chat** - No external channels required
- 📱 **Optional channels** - Email, X/Twitter, Telegram (Part 2)

---

## 📖 Guide Structure

The setup guide has **two parts**:

| Part | What You Get | Time |
|------|--------------|------|
| **Part 1: Core Setup** | Private Clawdbot server, secure SSH access, web chat | ~30-60 min |
| **Part 2: Digital Twin** | Email, GitHub, X accounts for autonomous AI | ~2-3 hours |

### Part 1 is all you need to start!

After Part 1, you can chat with Clawdbot via:
- ✅ SSH tunnel + built-in web chat
- ✅ CLI (`clawdbot chat` over SSH)
- ✅ VS Code + Roo Code extension
- ✅ Tailscale VPN (optional)

### Part 2 adds external communication

If you want Clawdbot to:
- 📧 Send/receive email
- 🐙 Push to GitHub repos
- 🐦 Post to X/Twitter
- 🌐 Maintain a public website

Then follow Part 2 to give Clawdbot its own identity.

### 🔐 Channel Security

| Channel | Security | Notes |
|---------|----------|-------|
| **SSH tunnel + Web chat** | ✅ Excellent | Only you have the PEM key |
| **Tailscale VPN** | ✅ Excellent | Encrypted mesh network |
| **X/Twitter DMs** | ✅ Good | Private, controlled account |
| **Email** | ✅ Good | Private inbox |
| **WhatsApp** | ⚠️ Risky | AI replies AS YOU — avoid |

## 🚀 Quick Start

### Prerequisites
- AWS Account (free tier works, but t2.small recommended)
- ~30 minutes for initial setup
- Optional: $1,000 AWS Activate credits ([apply here](https://aws.amazon.com/activate/))

### Steps

```bash
# 1. Clone this repo
git clone https://github.com/mayurjobanputra/Moltbot-Clawdbot-on-aws.git
cd Moltbot-Clawdbot-on-aws

# 2. Copy credentials template
cp credentials-template.md credentials.md

# 3. Follow the main guide
# Open CLAWDBOT_AWS_SETUP.md and follow step by step
```

### Or let AI do it for you:
```
"Hey Claude/GPT/Gemini, I want to set up Clawdbot on AWS. 
Here's a guide: [paste CLAWDBOT_AWS_SETUP.md contents]
Help me go through it step by step."
```

---

## 📁 Repository Structure

| File | Description |
|------|-------------|
| [`CLAWDBOT_AWS_SETUP.md`](CLAWDBOT_AWS_SETUP.md) | **Main guide** - Complete step-by-step instructions |
| [`credentials-template.md`](credentials-template.md) | Template for tracking your credentials (safe to commit) |
| `credentials.md` | Your actual credentials (gitignored - never commit!) |

---

## 🔌 Supported LLM Providers

This guide covers setting up multiple AI backends:

| Provider | Model Examples | Auth Method |
|----------|---------------|-------------|
| **AWS Bedrock** | Claude Opus 4.5, Sonnet, Haiku | AWS IAM credentials |
| **Google AI** | Gemini Pro, Flash | API Key |
| **Anthropic** | Claude (direct API) | API Key |
| **OpenAI** | GPT-4, GPT-4o | API Key |
| **OpenRouter** | Any model | API Key |

### AWS Bedrock (Recommended for AWS users)
- ✅ Uses your AWS credits ($1,000 Activate!)
- ✅ No separate API key needed
- ✅ Claude Opus 4.5, Sonnet 4.5, Haiku available
- ⚠️ Requires `us.` prefix for newer models (see guide)

---

## 🔒 Security

This setup follows security best practices based on [community audit findings](https://x.com/0xSammy/status/2015562918151020593):

| Security Feature | Status |
|-----------------|--------|
| Gateway bound to localhost | ✅ |
| SSH tunnel required for access | ✅ |
| Strong auth tokens | ✅ |
| No exposed ports (except SSH) | ✅ |
| Credential isolation | ✅ |

**⚠️ Never expose port 18789 to the internet!**

---

## 💰 Cost Estimate

| Component | Monthly Cost |
|-----------|-------------|
| EC2 t2.small | ~$17 |
| EBS 20GB | ~$2 |
| Elastic IP | Free (when attached) |
| **Total** | **~$19/month** |

With $1,000 AWS Activate credits: **~4+ years free!**

---

## 🛠️ Troubleshooting

Common issues and solutions are documented in the guide:

- **"Unknown model" errors** → Use `models.providers` config (not just `agents.defaults`)
- **"Missing auth" for Bedrock** → Create systemd drop-in file for AWS credentials
- **Gateway won't start** → Check config validation with `clawdbot doctor --fix`
- **Can't SSH** → Verify security group rules and PEM permissions

See the [Troubleshooting section](CLAWDBOT_AWS_SETUP.md#troubleshooting) in the main guide.

---

## 🤝 Contributing

Found an issue or have an improvement? PRs welcome!

1. Fork the repo
2. Create a feature branch
3. Make your changes
4. Submit a PR

---

## 📚 Resources

- [Clawdbot Documentation](https://docs.clawd.bot/)
- [AWS Bedrock Docs](https://docs.aws.amazon.com/bedrock/)
- [Crabwalk - Clawdbot Monitor](https://github.com/luccast/crabwalk)
- [Memory Features Tip (X Post)](https://x.com/mayloidy/status/1884451877418135613) - Enable memory flush & session search
- [Security Discussion (0xSammy)](https://x.com/0xSammy/status/2015562918151020593)
- [Daniel Miessler's Security Checklist](https://x.com/DanielMiessler/status/2015865548714975475)

---

## ⚠️ Important - Never Commit These:

```
❌ .pem files (SSH keys)
❌ API keys  
❌ credentials.md
❌ ~/.clawdbot/ contents
❌ AWS access keys
```

All sensitive files are in `.gitignore` for your protection.

---

## 📜 License

MIT License - Use freely, contribute back!

---

<details>
<summary>👤 Personal Project Notes (mayur.ai)</summary>

This repo was originally created for my personal AI assistant setup at mayur.ai:

**My Project Goals:**
1. ✅ Phase 1: Run Clawdbot on AWS VPS
2. ✅ Phase 2: Security lockdown
3. ⏳ Phase 3: Clawdbot identity (SES email + GitHub account)
4. ⏳ Phase 4: Public website on Vercel (separate from EC2)
5. ⏳ Phase 5: Crabwalk monitoring

**Architecture Decision:** Keep Clawdbot EC2 completely private. Public website hosted separately on Vercel. Clawdbot updates website via git push.

</details>

---

*Built with 🦞 by the community. Star ⭐ if this helped you!*

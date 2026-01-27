# mayur.ai - ClawdBot on AWS

Personal AI assistant infrastructure running [ClawdBot](https://docs.clawd.bot/) on AWS EC2.

## 🎯 Project Goals

1. **Phase 1**: Run ClawdBot as a personal AI assistant on AWS VPS
2. **Phase 2**: Lock down security so only I can access it
3. **Phase 3**: Build a VR avatar website on the same VPS
4. **Phase 4**: Serve the website via my domain (mayur.ai)
5. **Phase 5**: Add [Crabwalk](https://github.com/luccast/crabwalk) monitoring

## 📁 Files

| File | Description |
|------|-------------|
| [CLAWDBOT_AWS_SETUP.md](CLAWDBOT_AWS_SETUP.md) | Main setup guide with step-by-step instructions |
| [credentials-template.md](credentials-template.md) | Template showing what credentials you'll need |
| `credentials.md` | Your actual credentials (gitignored) |

## 🔒 Security First!

This setup prioritizes security based on [community findings](https://x.com/0xSammy/status/2015562918151020593):

- ✅ Gateway bound to `localhost` only (not exposed to internet)
- ✅ Access via SSH tunnel only
- ✅ SSH restricted to specific IP addresses
- ✅ Strong auth tokens
- ✅ Security audit commands included

## 🚀 Quick Start

1. Clone this repo
2. Copy `credentials-template.md` to `credentials.md`
3. Follow [CLAWDBOT_AWS_SETUP.md](CLAWDBOT_AWS_SETUP.md) step by step
4. Fill in `credentials.md` as you go

## 📚 References

- [ClawdBot Documentation](https://docs.clawd.bot/start/getting-started)
- [Crabwalk - ClawdBot Monitor](https://github.com/luccast/crabwalk)
- [Security Discussion](https://x.com/0xSammy/status/2015562918151020593)

## ⚠️ Important

**Never commit:**
- `.pem` files
- API keys
- `credentials.md`
- Anything in `.clawdbot/`

These are all in `.gitignore` for your protection.

---

*Building in public while keeping secrets private* 🔐

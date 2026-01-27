# X Thread: ClawdBot AWS Setup

## Thread 1/5 (Main post)

🤖 Just set up @ClawdBot on an AWS VPS — the RIGHT way.

After seeing @0xSammy's report about 923 exposed gateways, I made sure mine is locked down TIGHT.

Full setup guide open-sourced 👇

🔗 github.com/mayurjobanputra/mayur-dot-ai-clawdbot-on-aws

---

## Thread 2/5

The architecture:

✅ Gateway bound to localhost ONLY
✅ Access via SSH tunnel (requires my PEM key)
✅ Port 18789 completely invisible to the internet
✅ Token auth with 48-char cryptographic random

Even if you have my IP, you can't reach ClawdBot.

---

## Thread 3/5

Security hardening based on @DanielMiessler's checklist:

🔒 `bind: loopback` - not 0.0.0.0
🔒 `chmod 600` on all config files
🔒 SSH-only access (no port exposed)
🔒 Strong random auth token

The 923 exposed gateways? They had `bind: all`. Don't be one of them.

---

## Thread 4/5

The SSH tunnel trick:

```
ssh -L 18789:127.0.0.1:18789 -N clawdbot &
```

This forwards my local port to EC2's localhost.

Result: I access http://localhost:18789 on my Mac, but the traffic goes through encrypted SSH to my VPS.

Clean. Secure. Simple.

---

## Thread 5/5

Setup cost: ~$17/month on t2.small (2GB RAM)

The free tier t2.micro doesn't have enough RAM — learned that the hard way 😅

Full guide with troubleshooting, security steps, and quick commands:

🔗 github.com/mayurjobanputra/mayur-dot-ai-clawdbot-on-aws

Star it if you find it useful! ⭐

---

## Alt: Single Post Version

🤖 Set up @ClawdBot on AWS VPS — secured with SSH tunnel after seeing 923 exposed gateways in the wild.

✅ Gateway: localhost only
✅ Access: SSH tunnel required  
✅ Token: 48-char random
✅ Port 18789: invisible to internet

Full guide (free): github.com/mayurjobanputra/mayur-dot-ai-clawdbot-on-aws

cc @DanielMiessler @0xSammy

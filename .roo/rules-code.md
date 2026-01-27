# Code Mode Instructions for This Project

## ⚠️ CRITICAL RULE - READ FIRST

**NEVER write actual IPs, tokens, API keys, or any real credentials in ANY file.**

This includes:
- Documentation files (*.md)
- Config examples
- Code comments
- Even example patterns in rules files like this one

## Before ANY file edit or creation:

1. **Check for sensitive data** - Never write real IPs, tokens, or keys
2. **Use placeholders** - Always use `<ELASTIC_IP>`, `YOUR_TOKEN`, etc.
3. **Verify .gitignore** - Ensure sensitive files are listed

## Pre-commit checklist:

```bash
# Before every commit, run:
git diff --staged | grep -E '(\d+\.\d+\.\d+\.\d+|[a-f0-9]{32,})' && echo "⚠️ POTENTIAL SENSITIVE DATA FOUND!" || echo "✅ Clean"
```

## Always use these placeholders:

| Type | Use This Placeholder |
|------|---------------------|
| IP Address | `<ELASTIC_IP>` or `<YOUR_IP>` |
| Gateway Token | `YOUR_GATEWAY_TOKEN` |
| API Key | `YOUR_API_KEY` |
| PEM content | `[PEM KEY CONTENT]` |

## Files to protect:

- `credentials.md` → Local only, never commit
- Any IP or token should go in credentials.md, not main docs

## The user's real credentials are stored in:
- `credentials.md` (gitignored, local only)

Never read from credentials.md and write to any other file.

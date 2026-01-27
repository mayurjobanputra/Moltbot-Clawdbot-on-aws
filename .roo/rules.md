# Project Rules for ClawdBot AWS Setup

## Security Rules - CRITICAL

### Never include in any file that gets committed:
- Real IP addresses (like X.X.X.X)
- Gateway tokens or auth tokens
- API keys (Anthropic, OpenAI, etc.)
- PEM key contents
- Passwords or secrets

### Use placeholders instead:
- `<ELASTIC_IP>` for IP addresses
- `YOUR_GATEWAY_TOKEN` for tokens  
- `YOUR_API_KEY` for API keys
- `<YOUR_PUBLIC_IP_HERE>` for user IPs

### Files that should NEVER be committed:
- credentials.md (contains real credentials)
- *.pem files
- .env files with real values
- Any file with actual secrets

### Before committing, always check:
1. Run `git diff --staged` to review changes
2. Search for IP patterns: `grep -E '\d+\.\d+\.\d+\.\d+' <file>`
3. Search for token patterns: `grep -E '[a-f0-9]{32,}' <file>`

## File Patterns

### Safe to commit:
- CLAWDBOT_AWS_SETUP.md (documentation with placeholders)
- README.md
- credentials-template.md (empty template)
- .gitignore
- .roo/* files

### Never commit:
- credentials.md
- Any file matching patterns in .gitignore

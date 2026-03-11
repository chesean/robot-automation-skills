# Security Check Skill

Scan project code for security vulnerabilities using Project CodeGuard rules.

## Usage
```
/security-check
```

## When To Use
Before committing code, uploading jobfiles to CXTA, or creating API integrations.

## What Gets Checked

1. **Hardcoded credentials**: passwords, API keys, tokens in code
2. **Command injection**: os.system(), shell=True, eval(), exec()
3. **Input validation**: device names, IPs, file paths validated
4. **Logging security**: no credentials in log messages
5. **API security**: REST API best practices
6. **SSL/TLS**: proper certificate validation

## Process

Scan all *.py and *.robot files (excluding venv/, .git/):
- Check for credential patterns (password=, api_key=, token=, AWS keys, GitHub tokens)
- Check for dangerous functions (os.system, shell=True, eval, exec)
- Verify lab.yaml is in .gitignore
- Report: CRITICAL (hardcoded creds) / HIGH (injection risk) / WARNING (review needed)

## Rules Reference
- See `.claude/codeguard/NETWORK_AUTOMATION_SECURITY.md` for full guide
- See `.claude/codeguard/QUICK_REFERENCE.md` for patterns

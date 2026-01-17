# Security Audit Quick Checklist

## Pre-Deployment Security Gate

Run through this checklist before any deployment. Each item is pass/fail.

---

## 🔴 CRITICAL (Block deployment if any fail)

### Secrets Exposure
```bash
# Run these commands in your repo root:

# Check for secrets in code
npx secretlint "**/*"
# OR
trufflehog filesystem .

# Check git history for leaked secrets
trufflehog git file://. --since-commit HEAD~100

# Verify .env is gitignored
cat .gitignore | grep -E "^\.env"
```
- [ ] No hardcoded API keys, passwords, or tokens
- [ ] .env files are gitignored
- [ ] No secrets in git history

### SQL Injection
```bash
# Find raw queries
grep -rn "query\|execute\|raw(" --include="*.js" --include="*.ts" --include="*.py" | grep -v node_modules

# Check for string concatenation in SQL
grep -rn "\`SELECT\|\"SELECT\|'SELECT" --include="*.js" --include="*.ts" --include="*.py" | grep -v node_modules
```
- [ ] All database queries use parameterized statements
- [ ] No string concatenation for SQL
- [ ] ORM used properly (no raw queries with user input)

### XSS Prevention
```bash
# Find dangerous patterns
grep -rn "innerHTML\|dangerouslySetInnerHTML\|v-html\|document.write\|\.html(" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx" --include="*.vue" | grep -v node_modules
```
- [ ] No innerHTML with user data
- [ ] No dangerouslySetInnerHTML without sanitization
- [ ] Output encoding on all user-generated content

### Authentication
```bash
# Check password hashing
grep -rn "bcrypt\|argon2\|scrypt" --include="*.js" --include="*.ts" --include="*.py" | head -5

# Red flags - weak hashing
grep -rn "md5\|sha1\|sha256" --include="*.js" --include="*.ts" --include="*.py" | grep -i password
```
- [ ] Passwords hashed with bcrypt/argon2/scrypt
- [ ] No MD5/SHA1/SHA256 for passwords
- [ ] Sessions invalidated on logout (server-side)

---

## 🟠 HIGH PRIORITY

### Authorization (IDOR)
```
Manual Test:
1. Create 2 test accounts (User A, User B)
2. As User A, note resource IDs (profile, documents, etc.)
3. As User B, try accessing User A's resources by ID
4. Any success = IDOR vulnerability
```
- [ ] Server verifies resource ownership on every request
- [ ] User can't access other users' data via ID manipulation
- [ ] Admin endpoints require admin role check

### Rate Limiting
```bash
# Quick test with curl
for i in {1..100}; do curl -s -o /dev/null -w "%{http_code}\n" https://yourapp.com/api/login; done | sort | uniq -c

# Should see 429 responses after threshold
```
- [ ] Login endpoint rate limited (5-10 attempts)
- [ ] API endpoints have rate limits
- [ ] Password reset rate limited
- [ ] Expensive operations limited

### CSRF Protection
```html
<!-- Save as test.html, open while logged into target app -->
<form action="https://yourapp.com/api/sensitive-action" method="POST">
  <input type="hidden" name="action" value="delete">
  <button>Test CSRF</button>
</form>

<!-- If it works, you're vulnerable -->
```
- [ ] CSRF tokens on all state-changing requests
- [ ] SameSite cookie attribute set (Strict or Lax)
- [ ] Origin/Referer validation for sensitive actions

### Security Headers
```bash
# Quick check
curl -I https://yourapp.com | grep -iE "content-security|x-frame|x-content-type|strict-transport|x-xss"

# Or use online tool
# https://securityheaders.com
```
- [ ] Content-Security-Policy set
- [ ] X-Frame-Options: DENY or SAMEORIGIN
- [ ] X-Content-Type-Options: nosniff
- [ ] Strict-Transport-Security enabled
- [ ] Referrer-Policy set

### CORS Configuration
```bash
# Check CORS headers
curl -H "Origin: https://evil.com" -I https://yourapp.com/api/endpoint

# FAIL if you see:
# Access-Control-Allow-Origin: * (with credentials)
# Access-Control-Allow-Origin: https://evil.com (reflected)
```
- [ ] No wildcard (*) with credentials
- [ ] Origin not blindly reflected
- [ ] Only necessary origins allowed

---

## 🟡 MEDIUM PRIORITY

### File Uploads
```
Manual Test:
1. Try uploading .php, .html, .svg files
2. Try filename: ../../../etc/passwd.jpg
3. Try double extension: file.php.jpg
4. Check if uploaded files are accessible directly
```
- [ ] File type validated server-side (not just extension)
- [ ] Files stored outside webroot
- [ ] Filenames sanitized (no path traversal)
- [ ] Content-Disposition: attachment for downloads
- [ ] Size limits enforced

### Error Handling
```bash
# Trigger errors and check response
curl -X POST https://yourapp.com/api/endpoint -d "invalid{json"
curl https://yourapp.com/api/nonexistent

# Should NOT see:
# - Stack traces
# - SQL errors
# - File paths
# - Version numbers
```
- [ ] Generic error messages in production
- [ ] No stack traces exposed
- [ ] No database errors leaked
- [ ] No file paths revealed

### Session Management
```
Manual Test:
1. Login and note session token
2. Logout
3. Try using old session token
4. If it works, sessions aren't properly invalidated
```
- [ ] Session destroyed on logout
- [ ] Session timeout configured
- [ ] Session ID regenerated after login
- [ ] Cookies: HttpOnly, Secure, SameSite

### Input Validation
```bash
# Test with malicious inputs in all forms:
# ' OR '1'='1' --
# <script>alert(1)</script>
# ${7*7}
# {{7*7}}
# $(whoami)
# -1
# 99999999999999999
# test@test.com\nBcc:attacker@evil.com
```
- [ ] All inputs validated server-side
- [ ] Type checking (numbers are numbers)
- [ ] Length limits on all text fields
- [ ] Enum values validated against allowed list

### Dependency Security
```bash
# JavaScript
npm audit
npx snyk test

# Python
pip-audit
safety check

# Go
govulncheck ./...

# Check for outdated packages
npm outdated
pip list --outdated
```
- [ ] No high/critical vulnerabilities
- [ ] Dependencies reasonably up-to-date
- [ ] Lock file committed

---

## 🟢 LOWER PRIORITY (But Still Important)

### Logging
- [ ] Auth events logged (success/failure)
- [ ] No passwords in logs
- [ ] No tokens/PII in logs
- [ ] Logs have timestamps and user IDs

### Infrastructure
```bash
# Check exposed paths
for path in .env .git/config backup.sql phpinfo.php server-status; do
  status=$(curl -s -o /dev/null -w "%{http_code}" https://yourapp.com/$path)
  echo "$path: $status"
done

# All should return 404 or 403
```
- [ ] .env not accessible
- [ ] .git not exposed
- [ ] Debug endpoints disabled
- [ ] Admin panels protected

### HTTPS
```bash
# Test SSL configuration
# https://www.ssllabs.com/ssltest/

# Or with curl
curl -vI https://yourapp.com 2>&1 | grep -E "SSL|TLS"
```
- [ ] TLS 1.2+ only
- [ ] Valid certificate
- [ ] HSTS enabled
- [ ] No mixed content

---

## 🛠️ AUTOMATED SCANNING

Run these tools for comprehensive scanning:

### OWASP ZAP (Free)
```bash
# Docker quick scan
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://yourapp.com

# Full scan (takes longer)
docker run -t owasp/zap2docker-stable zap-full-scan.py -t https://yourapp.com
```

### Nuclei (Free)
```bash
# Install
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Scan
nuclei -u https://yourapp.com -t cves/ -t vulnerabilities/
```

### Semgrep (Free)
```bash
# Install
pip install semgrep

# Scan codebase
semgrep --config=auto .
semgrep --config=p/security-audit .
```

### SQLMap (Free)
```bash
# Test specific endpoint
sqlmap -u "https://yourapp.com/api/search?q=test" --batch --level=2

# Test form
sqlmap -u "https://yourapp.com/login" --data="user=test&pass=test" --batch
```

---

## 📝 DOCUMENTATION CHECKLIST

- [ ] Security contact documented (security.txt)
- [ ] Incident response plan exists
- [ ] Password policy documented
- [ ] Data retention policy documented
- [ ] Third-party security requirements documented

---

## 🔄 ONGOING SECURITY

### Weekly
- [ ] Check dependency vulnerabilities
- [ ] Review access logs for anomalies
- [ ] Check for new CVEs affecting your stack

### Monthly
- [ ] Rotate API keys/secrets
- [ ] Review user permissions
- [ ] Update dependencies
- [ ] Run automated scan

### Quarterly
- [ ] Full security audit
- [ ] Review third-party integrations
- [ ] Update security documentation
- [ ] Security training for team

---

## ⚡ ONE-COMMAND AUDIT SCRIPT

Save as `security-check.sh`:

```bash
#!/bin/bash

echo "🔐 SECURITY AUDIT STARTING..."
echo "=============================="

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# 1. Secrets scan
echo -e "\n${YELLOW}[1/7] Scanning for secrets...${NC}"
if command -v trufflehog &> /dev/null; then
    trufflehog filesystem . --no-update 2>/dev/null | head -20
else
    grep -rn "password\|api_key\|secret" --include="*.js" --include="*.ts" --include="*.py" --include="*.env*" 2>/dev/null | grep -v node_modules | head -10
fi

# 2. Dependency check
echo -e "\n${YELLOW}[2/7] Checking dependencies...${NC}"
if [ -f "package.json" ]; then
    npm audit --json 2>/dev/null | jq -r '.metadata.vulnerabilities'
fi
if [ -f "requirements.txt" ]; then
    pip-audit 2>/dev/null | head -20
fi

# 3. Dangerous patterns
echo -e "\n${YELLOW}[3/7] Checking dangerous code patterns...${NC}"
echo "innerHTML/dangerouslySetInnerHTML usage:"
grep -rn "innerHTML\|dangerouslySetInnerHTML" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx" 2>/dev/null | grep -v node_modules | wc -l

echo "eval/exec usage:"
grep -rn "eval(\|exec(" --include="*.js" --include="*.ts" --include="*.py" 2>/dev/null | grep -v node_modules | wc -l

# 4. SQL injection risks
echo -e "\n${YELLOW}[4/7] Checking SQL query patterns...${NC}"
grep -rn "query\|execute" --include="*.js" --include="*.ts" --include="*.py" 2>/dev/null | grep -v node_modules | grep -E "\\\$\{|\+.*SELECT|%s" | head -10

# 5. Hardcoded values
echo -e "\n${YELLOW}[5/7] Checking for hardcoded credentials...${NC}"
grep -rn "password.*=.*['\"]" --include="*.js" --include="*.ts" --include="*.py" 2>/dev/null | grep -v node_modules | grep -v "password.*=.*['\"]['\"]" | head -10

# 6. Console.log in production
echo -e "\n${YELLOW}[6/7] Checking debug statements...${NC}"
echo "console.log statements:"
grep -rn "console.log" --include="*.js" --include="*.ts" 2>/dev/null | grep -v node_modules | wc -l

# 7. Env check
echo -e "\n${YELLOW}[7/7] Checking .env handling...${NC}"
if git ls-files --error-unmatch .env 2>/dev/null; then
    echo -e "${RED}WARNING: .env is tracked in git!${NC}"
else
    echo -e "${GREEN}.env is properly gitignored${NC}"
fi

echo -e "\n=============================="
echo "🔐 BASIC SCAN COMPLETE"
echo "Run OWASP ZAP for comprehensive scanning"
```

Make executable: `chmod +x security-check.sh`

---

## 📚 RESOURCES

- **OWASP Top 10**: https://owasp.org/Top10/
- **OWASP Cheat Sheets**: https://cheatsheetseries.owasp.org/
- **CWE Database**: https://cwe.mitre.org/
- **HackTricks**: https://book.hacktricks.wiki/
- **PortSwigger Web Academy**: https://portswigger.net/web-security

---

*Last updated: Security never sleeps*

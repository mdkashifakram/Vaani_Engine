# Security Audit Master Prompt for AI-Assisted Development

## How to Use This Prompt

Copy the relevant sections into your AI assistant (Claude, GPT, etc.) along with your codebase or specific files. Run through each phase systematically.

---

# 🔐 MASTER SECURITY AUDIT PROMPT

## Initial Context (Paste This First)

```
You are a senior security engineer conducting a comprehensive security audit. Your role is to identify vulnerabilities, misconfigurations, and security anti-patterns in the codebase provided.

Be thorough, specific, and actionable. For each issue found:
1. Identify the exact file and line number
2. Explain the vulnerability and attack vector
3. Rate severity (Critical/High/Medium/Low)
4. Provide the exact fix with code

Do not be reassuring. Be direct about problems. Assume attackers are sophisticated.

Tech Stack Context: [FILL IN YOUR STACK - e.g., "Next.js 14, PostgreSQL, Prisma, NextAuth, deployed on Vercel"]
```

---

## PHASE 1: RECONNAISSANCE & ARCHITECTURE REVIEW

### Prompt 1.1: Architecture Security Review

```
Analyze the project structure and identify:

1. **Entry Points**: All routes, API endpoints, webhooks, file upload handlers
2. **Data Flow**: How user input moves through the system
3. **Trust Boundaries**: Where authenticated vs unauthenticated code runs
4. **External Integrations**: Third-party APIs, payment processors, auth providers
5. **Sensitive Data Locations**: Where PII, credentials, tokens are stored/processed

Create a security-focused architecture diagram showing:
- Attack surface areas
- Data flow with trust boundaries marked
- Potential lateral movement paths

List all files that handle:
- Authentication/Authorization
- User input processing
- Database queries
- File operations
- External API calls
- Cryptographic operations
```

### Prompt 1.2: Dependency Audit

```
Analyze package.json / requirements.txt / go.mod (whichever applies):

1. List all dependencies with known CVEs (check against NVD database)
2. Identify abandoned packages (no updates in 2+ years)
3. Flag packages with excessive permissions or install scripts
4. Check for typosquatting risks (similar names to popular packages)
5. Identify unnecessary dependencies that increase attack surface

For each vulnerable dependency:
- CVE ID and severity
- Attack vector
- Is it exploitable in our usage context?
- Upgrade path or alternative package

Run mentally: npm audit / pip-audit / go mod verify
```

---

## PHASE 2: INJECTION VULNERABILITIES

### Prompt 2.1: SQL Injection Audit

```
Search the entire codebase for SQL injection vulnerabilities:

1. Find ALL database queries (raw SQL, ORM usage, query builders)
2. For each query, trace back: Is ANY part constructed from user input?
3. Check for:
   - String concatenation in queries
   - Template literals with user data
   - Dynamic column/table names from user input
   - ORDER BY / LIMIT from user input
   - JSON/JSONB queries with user data
   - LIKE clauses with unescaped wildcards

Flag as CRITICAL:
- Any raw SQL with string interpolation
- Any query where user input isn't parameterized

Provide fix for each:
- Show the vulnerable code
- Show the parameterized version
```

### Prompt 2.2: XSS (Cross-Site Scripting) Audit

```
Identify all XSS vulnerabilities:

1. **Stored XSS**: Find where user input is:
   - Saved to database
   - Later rendered in HTML
   - Check: Is it sanitized before storage AND escaped on output?

2. **Reflected XSS**: Find where:
   - URL parameters are rendered
   - Search queries are displayed
   - Error messages include user input

3. **DOM XSS**: Find client-side code using:
   - innerHTML, outerHTML
   - document.write()
   - eval(), Function(), setTimeout/setInterval with strings
   - jQuery html(), append() with user data
   - React's dangerouslySetInnerHTML
   - v-html in Vue
   - [innerHTML] in Angular

4. Check all:
   - User profile fields (name, bio, etc.)
   - Comments/reviews
   - File names displayed
   - URL redirects

For each finding, show:
- The vulnerable code path
- Example payload that would execute
- The proper sanitization/escaping fix
```

### Prompt 2.3: Command Injection Audit

```
Find command injection vulnerabilities:

1. Search for:
   - exec(), spawn(), execSync(), execFile()
   - system(), popen(), subprocess
   - Shell commands via any method

2. For each occurrence:
   - Is ANY argument derived from user input?
   - Is input validated against allowlist?
   - Are shell metacharacters escaped?

3. Check for indirect command injection via:
   - Filename arguments
   - Environment variables from user input
   - Configuration values from user input

Red flags:
- Template strings in shell commands
- User input in any exec/spawn argument
- Path traversal possibilities

Provide safer alternatives (spawn with array arguments, avoiding shell entirely)
```

### Prompt 2.4: Other Injection Types

```
Check for these injection vulnerabilities:

1. **LDAP Injection**: Search for LDAP queries with user input
2. **XML Injection / XXE**: XML parsing with external entities enabled
3. **Template Injection**: User input in server-side templates (Jinja2, EJS, Handlebars)
4. **Header Injection**: User input in HTTP headers (especially redirects, Set-Cookie)
5. **Log Injection**: Unsanitized input written to logs (can forge log entries)
6. **Email Header Injection**: User input in email headers (To, CC, Subject)
7. **NoSQL Injection**: MongoDB queries with $where or user-controlled operators

For each type found:
- Show the vulnerable code
- Demonstrate the attack
- Provide the fix
```

---

## PHASE 3: AUTHENTICATION & SESSION MANAGEMENT

### Prompt 3.1: Authentication Security Audit

```
Audit the entire authentication system:

1. **Password Security**:
   - What hashing algorithm? (Must be bcrypt/argon2/scrypt, NOT MD5/SHA1/SHA256)
   - What's the work factor/cost?
   - Are passwords ever logged or stored in plain text anywhere?
   - Is there a minimum password length/complexity?

2. **Login Flow**:
   - Is there rate limiting? What are the limits?
   - Account lockout after failed attempts?
   - Are login errors generic? (Don't reveal if user exists)
   - Is there CAPTCHA after failures?
   - Is the login form served over HTTPS?

3. **Password Reset**:
   - How are reset tokens generated? (Must be cryptographically random)
   - Token expiration time?
   - One-time use enforcement?
   - Is the old password invalidated immediately?
   - Can reset link be reused?

4. **Multi-Factor Authentication**:
   - Is MFA available/enforced?
   - TOTP implementation correct?
   - Backup codes generated securely?
   - Can MFA be bypassed?

5. **OAuth/SSO** (if applicable):
   - State parameter validated?
   - Tokens verified server-side?
   - Proper scope restrictions?

List ALL authentication-related files and their security status.
```

### Prompt 3.2: Session Management Audit

```
Audit session handling:

1. **Session Token Security**:
   - How are session IDs generated? (Check for crypto.randomBytes or equivalent)
   - Token length and entropy?
   - Are tokens in URL? (BAD - they leak in referrer headers)

2. **Cookie Security**:
   - HttpOnly flag set?
   - Secure flag set?
   - SameSite attribute? (Should be Strict or Lax)
   - Proper domain/path restrictions?
   - Reasonable expiration?

3. **Session Lifecycle**:
   - Is session destroyed on logout? (Server-side, not just cookie deletion)
   - Session timeout for inactivity?
   - Absolute session timeout?
   - Session regeneration after login? (Prevents fixation)
   - Session regeneration after privilege change?

4. **Concurrent Sessions**:
   - Limit on concurrent sessions?
   - Can user view/revoke other sessions?
   - Is there "logout everywhere" functionality?

5. **Session Fixation**:
   - Try: Set a session ID before login, check if same ID works after login
   
Find all session-related code and verify each point.
```

### Prompt 3.3: Authorization (Access Control) Audit

```
Audit authorization throughout the application:

1. **IDOR (Insecure Direct Object Reference)**:
   - List ALL endpoints that access resources by ID
   - For EACH: Is ownership/permission verified server-side?
   - Check: /api/users/{id}, /api/documents/{id}, /api/orders/{id}, etc.
   - Can user A access user B's resources by changing the ID?

2. **Privilege Escalation**:
   - Can users modify their own role/permissions?
   - Are admin functions protected beyond just UI hiding?
   - Check for mass assignment (can user send {isAdmin: true}?)
   - Are role checks done on every request or just once?

3. **Path Traversal**:
   - Any file operations with user input?
   - Can ../../ be used to escape intended directories?
   - Are file paths validated/canonicalized?

4. **Function Level Access Control**:
   - Are all admin endpoints protected?
   - Are there any endpoints missing auth middleware?
   - Check for debug/test endpoints in production

Map out: Every endpoint → Required permission → Current enforcement
Flag any endpoint where enforcement is missing or client-side only.
```

---

## PHASE 4: DATA PROTECTION

### Prompt 4.1: Sensitive Data Exposure Audit

```
Find all sensitive data handling issues:

1. **API Response Audit**:
   - For EVERY API endpoint, what data is returned?
   - Are password hashes ever exposed?
   - Are internal IDs exposed that enable enumeration?
   - Is PII returned when not needed?
   - Are there /debug or /admin endpoints exposing data?

2. **Logging Audit**:
   - What gets logged?
   - Are passwords/tokens/PII in logs?
   - Are logs accessible to too many people?
   - Search for: console.log, logger., print(, logging.

3. **Error Messages**:
   - Do errors expose stack traces in production?
   - Do database errors leak schema/query information?
   - Do file errors leak path information?

4. **Client-Side Exposure**:
   - Are there secrets in frontend JavaScript?
   - Check: API keys, internal URLs, debug flags
   - Search frontend bundle for: apiKey, secret, password, token, private

5. **Transmission Security**:
   - Is ALL traffic over HTTPS?
   - Is HSTS enabled?
   - Are there mixed content issues?

6. **Storage Security**:
   - Is sensitive data encrypted at rest?
   - How are encryption keys managed?
   - Is there any sensitive data in localStorage/sessionStorage?
```

### Prompt 4.2: Secrets Management Audit

```
Find all hardcoded secrets and credentials:

1. **Search the entire codebase for**:
   - API keys, tokens, passwords
   - Connection strings
   - Private keys, certificates
   - JWT secrets
   - Encryption keys

   Search patterns:
   - "password", "passwd", "pwd"
   - "secret", "apikey", "api_key"
   - "token", "auth", "bearer"
   - "private_key", "privateKey"
   - Base64 encoded strings (potential encoded secrets)
   - High-entropy strings

2. **Check**:
   - .env files committed to repo?
   - .env.example with real values?
   - Secrets in docker-compose.yml?
   - Secrets in CI/CD configs?
   - Secrets in Terraform/IaC files?

3. **Git History**:
   - Were secrets ever committed and "removed"? (They're still in history)

4. **Environment Variables**:
   - Are secrets passed as command line args? (Visible in process list)
   - Are secrets in Dockerfile ENV instructions? (Baked into image)

For each secret found:
- Location
- Type of secret
- Rotation required? (assume yes if ever exposed)
- Proper storage recommendation
```

---

## PHASE 5: API SECURITY

### Prompt 5.1: API Security Audit

```
Comprehensive API security audit:

1. **Rate Limiting**:
   - Is rate limiting implemented?
   - Per-endpoint or global?
   - By IP, user, or both?
   - What are the limits? Are they appropriate?
   - What happens when limit exceeded? (Should be 429)
   - Can rate limiting be bypassed (X-Forwarded-For spoofing)?

2. **Input Validation**:
   - Is ALL input validated server-side?
   - Are there length limits on all text fields?
   - Are numeric inputs bounded?
   - Are enums validated against allowed values?
   - Is JSON schema validation used?
   - Are arrays bounded to prevent DoS?

3. **CORS Configuration**:
   - What is Access-Control-Allow-Origin set to?
   - Is it wildcard (*) with credentials? (CRITICAL vulnerability)
   - Does it reflect Origin header without validation?
   - Are only necessary methods/headers allowed?

4. **API Versioning & Documentation**:
   - Are old API versions still active with known vulnerabilities?
   - Is API documentation exposing sensitive endpoints?
   - Are debug endpoints disabled in production?

5. **Request Validation**:
   - Content-Type validated?
   - File upload limits enforced?
   - Request body size limits?

For each API endpoint, document:
- Authentication required?
- Authorization checks?
- Rate limiting applied?
- Input validation?
- Output sanitization?
```

### Prompt 5.2: CSRF Protection Audit

```
Audit Cross-Site Request Forgery protections:

1. **CSRF Token Implementation**:
   - Are CSRF tokens implemented for state-changing requests?
   - Token generation: Is it cryptographically random?
   - Token validation: Is it checked server-side on every request?
   - Token binding: Is it tied to user session?
   - Token storage: Cookie with appropriate flags? Hidden form field?

2. **Endpoints to Check**:
   - ALL POST/PUT/DELETE endpoints
   - Any GET endpoint that changes state (BAD pattern)
   - Password change, email change
   - Payment/purchase endpoints
   - Account deletion
   - Settings changes

3. **SameSite Cookie Analysis**:
   - Is SameSite attribute set on session cookies?
   - What value? (Strict > Lax > None)
   - If None, is Secure flag also set?

4. **Test Scenarios**:
   - Can I submit a form from a different origin to state-changing endpoints?
   - Does the server accept requests without the CSRF token?
   - Does the server accept requests with an invalid CSRF token?
   - Can the token be reused?

For each vulnerable endpoint:
- Show the endpoint and method
- Explain the attack scenario
- Provide the fix (token implementation)
```

---

## PHASE 6: FILE HANDLING

### Prompt 6.1: File Upload Security Audit

```
Audit all file upload functionality:

1. **Find All Upload Handlers**:
   - List every endpoint that accepts file uploads
   - What file types are allowed?
   - What's the size limit?

2. **Validation Checks**:
   - Is file extension validated? (CLIENT-SIDE ONLY IS NOT ENOUGH)
   - Is MIME type validated server-side?
   - Is file content/magic bytes validated?
   - Are there tests for extension bypass (.php.jpg, .php%00.jpg)?

3. **Storage Security**:
   - Where are files stored? (Must be OUTSIDE webroot)
   - Are filenames sanitized? (Remove path traversal, special chars)
   - Are files renamed to random names?
   - Can uploaded files be executed as code?

4. **Access Control**:
   - Can users access other users' files?
   - Are there direct URLs to uploaded files? (Should require auth)
   - Content-Disposition: attachment set? (Prevents browser execution)
   - X-Content-Type-Options: nosniff set?

5. **Dangerous File Types**:
   - Can .html/.svg files be uploaded and served? (XSS vector)
   - Can .php/.jsp/.aspx files be uploaded?
   - Are polyglot files detected? (Valid image that's also valid HTML)

6. **Resource Limits**:
   - Zip bomb protection?
   - Image bomb protection? (Small file, huge dimensions)
   - Total storage quota per user?

For each upload handler:
- Current validation
- Vulnerabilities found
- Complete secure implementation
```

### Prompt 6.2: File Download/Read Security Audit

```
Audit file download and read functionality:

1. **Path Traversal**:
   - Any endpoint that reads files based on user input?
   - Can ../../ or ..%2F escape intended directory?
   - Is path canonicalized before use?
   - Are symlinks followed? (Can escape restrictions)

2. **Access Control**:
   - Can users download files they don't own?
   - Are sensitive files protected? (.env, config files, source code)
   - Is there logging of file access?

3. **Information Disclosure**:
   - Can backup files be accessed? (.bak, .old, ~)
   - Can version control files be accessed? (.git)
   - Are directory listings enabled?
   - Can error messages reveal file paths?

4. **SSRF via File Operations**:
   - Can file URLs (file://) be specified?
   - Can internal URLs be accessed?
   - Is there URL validation for remote file operations?

Test these paths:
- /api/files/../../../etc/passwd
- /api/download?file=....//....//etc/passwd
- /api/read?path=.env
- /api/files?name=..%252f..%252fetc/passwd (double encoding)
```

---

## PHASE 7: INFRASTRUCTURE & CONFIGURATION

### Prompt 7.1: Security Headers Audit

```
Audit HTTP security headers:

Check for these headers in server responses:

1. **Content-Security-Policy**:
   - Is it set?
   - Does it allow 'unsafe-inline' or 'unsafe-eval'? (Weakens protection)
   - Are sources restricted appropriately?

2. **X-Content-Type-Options**: Should be "nosniff"

3. **X-Frame-Options**: Should be "DENY" or "SAMEORIGIN"

4. **Strict-Transport-Security** (HSTS):
   - Is it set?
   - max-age at least 31536000?
   - includeSubDomains?
   - preload?

5. **X-XSS-Protection**: "1; mode=block" (legacy but still useful)

6. **Referrer-Policy**: "strict-origin-when-cross-origin" or stricter

7. **Permissions-Policy**: Restrict unnecessary browser features

8. **Cache-Control**: Sensitive pages shouldn't be cached

For each missing/weak header:
- Current value (or "missing")
- Recommended value
- Implementation code for the framework
```

### Prompt 7.2: Server Configuration Audit

```
Audit server and deployment configuration:

1. **Exposed Sensitive Paths**:
   Check if accessible:
   - /.env
   - /.git/config
   - /config.php, /settings.py
   - /backup/, /bak/
   - /phpinfo.php
   - /server-status, /server-info
   - /debug/, /test/
   - /api/docs (should require auth in prod)
   - /graphql/playground
   - /admin/ (should require auth)
   - /actuator/ (Spring Boot)
   - /__debug__/ (Django)

2. **Server Information Disclosure**:
   - Server header exposing version?
   - X-Powered-By header?
   - Error pages revealing stack/version?

3. **TLS Configuration**:
   - TLS 1.2+ only?
   - Strong cipher suites?
   - Valid certificate?
   - Certificate pinning (mobile apps)?

4. **Environment Separation**:
   - Debug mode disabled in production?
   - Production database separate from dev?
   - Test credentials removed?
   - Staging accessible without auth?

5. **Container Security** (if applicable):
   - Running as root? (BAD)
   - Unnecessary capabilities?
   - Secrets in image?
   - Base image up to date?

For each issue:
- What was found
- Risk
- Remediation
```

---

## PHASE 8: BUSINESS LOGIC & ADVANCED ATTACKS

### Prompt 8.1: Business Logic Audit

```
Audit business logic for security flaws:

1. **Payment/Financial Logic**:
   - Is price calculated server-side? (Never trust client)
   - Can discounts be applied multiple times?
   - Can negative quantities result in credits?
   - Race condition in payment processing?
   - Are transactions idempotent?

2. **Workflow Bypass**:
   - Can multi-step processes be skipped?
   - Can users submit final step without prerequisites?
   - Can email verification be bypassed?
   - Can approval workflows be circumvented?

3. **Rate/Limit Abuse**:
   - Can trial periods be extended?
   - Can free tier limits be bypassed?
   - Can referral bonuses be self-referred?
   - Can votes/likes be inflated?

4. **Time-Based Issues**:
   - Are timestamps validated server-side?
   - Can expired tokens/coupons still be used?
   - Race conditions in time-sensitive operations?

5. **Trust Boundary Violations**:
   - Does admin trust user-submitted data too much?
   - Can users inject content seen by admins?
   - Can users influence emails sent to others?

For each logic flaw:
- Describe the normal flow
- Describe the exploit
- Business impact
- Fix
```

### Prompt 8.2: SSRF Audit

```
Audit for Server-Side Request Forgery:

1. **Find All URL Inputs**:
   - Webhook URLs
   - Image/URL preview features
   - PDF generators with URL input
   - Import from URL features
   - Link unfurling
   - API callbacks

2. **For Each, Test**:
   - Can I specify http://127.0.0.1?
   - Can I specify http://localhost?
   - Can I specify internal IPs? (10.x, 172.16.x, 192.168.x)
   - Can I specify http://169.254.169.254? (Cloud metadata)
   - Can I use DNS rebinding?
   - Can I use URL shorteners to redirect?
   - Can I use non-HTTP protocols? (file://, gopher://, dict://)

3. **Bypasses to Try**:
   - Decimal IP: http://2130706433 (127.0.0.1)
   - IPv6: http://[::1]
   - DNS pointing to 127.0.0.1
   - URL encoding
   - Redirect chains

4. **Impact Assessment**:
   - Can I read cloud metadata credentials?
   - Can I scan internal ports?
   - Can I access internal services?
   - Can I exfiltrate data?

For each SSRF vector:
- The vulnerable endpoint
- POC request
- What can be accessed
- Mitigation (allowlist, don't follow redirects, etc.)
```

### Prompt 8.3: Race Condition Audit

```
Audit for race conditions (TOCTOU bugs):

1. **Financial Operations**:
   - Balance checks before transfers
   - Inventory checks before purchase
   - Coupon usage checks

2. **Limit Checks**:
   - Rate limit checks
   - Quota checks
   - One-time use token checks

3. **State Changes**:
   - User status changes
   - Permission changes
   - File operations

For each potential race condition:

1. Identify the vulnerable sequence:
   - Check → Action pattern
   - Read → Modify → Write pattern

2. Determine exploitability:
   - Can the check and action be separated by concurrent requests?
   - What's the time window?

3. Test methodology:
   - Send 10-100 concurrent identical requests
   - Use tools like Turbo Intruder or race-the-web

4. Fix patterns:
   - Database transactions with proper isolation
   - Optimistic locking
   - Atomic operations
   - Mutex/semaphores for critical sections

Document: The vulnerable code path, how to exploit, and the atomic fix.
```

---

## PHASE 9: CRYPTOGRAPHY

### Prompt 9.1: Cryptography Audit

```
Audit all cryptographic implementations:

1. **Password Hashing**:
   - Algorithm used? (MUST be bcrypt, argon2, or scrypt)
   - Cost factor/work factor?
   - Salt generated properly?
   - FAIL if: MD5, SHA1, SHA256 (without proper KDF)

2. **Token Generation**:
   - Session tokens: crypto.randomBytes or equivalent?
   - Reset tokens: Sufficient entropy?
   - API keys: Proper randomness?
   - FAIL if: Math.random(), predictable seeds, timestamps

3. **Encryption**:
   - Algorithm: AES-256-GCM preferred
   - Mode: ECB is ALWAYS wrong
   - IV/Nonce: Random, never reused?
   - Key management: How are keys stored/rotated?

4. **JWT Security**:
   - Algorithm: RS256 preferred, HS256 acceptable
   - FAIL if: Algorithm "none" accepted
   - FAIL if: Secret key is weak/guessable
   - Is signature actually verified?
   - Are claims (exp, iat, aud) validated?

5. **TLS/SSL**:
   - Version: TLS 1.2+ only
   - Cipher suites: Strong only
   - Certificate validation: Not disabled anywhere?

6. **Hashing for Integrity**:
   - SHA-256 minimum for checksums
   - HMAC for authentication codes

Search for:
- crypto, bcrypt, jwt, hash, encrypt, random
- md5, sha1, des, rc4, ecb (red flags)

For each finding:
- Current implementation
- Security issue
- Correct implementation
```

---

## PHASE 10: THIRD-PARTY INTEGRATIONS

### Prompt 10.1: Third-Party Security Audit

```
Audit all third-party integrations:

1. **OAuth Implementations**:
   - State parameter used and validated?
   - Redirect URI strictly validated?
   - Tokens exchanged server-side?
   - Token storage secure?
   - Scopes minimized?

2. **Payment Processors**:
   - Server-side price calculation?
   - Webhook signatures verified?
   - PCI compliance (card data never touches your server)?
   - Idempotency keys used?
   - Error handling doesn't leak card info?

3. **Webhooks Received**:
   - Signatures verified?
   - Replay protection?
   - HTTPS only?
   - IP allowlisting?
   - Timeout handling?

4. **APIs Consumed**:
   - API keys stored securely?
   - Principle of least privilege?
   - Error handling doesn't expose keys?
   - Rate limiting handled gracefully?
   - Response validation?

5. **CDN/Third-Party Scripts**:
   - Subresource Integrity (SRI) used?
   - What scripts are loaded?
   - Could they be compromised?

For each integration:
- Security measures in place
- Missing protections
- Recommendations
```

---

## PHASE 11: DENIAL OF SERVICE

### Prompt 11.1: DoS Vulnerability Audit

```
Audit for Denial of Service vulnerabilities:

1. **Resource Exhaustion**:
   - API rate limiting comprehensive?
   - File upload size limits?
   - Request body size limits?
   - Query complexity limits (GraphQL)?
   - Pagination limits (can't request 1 million items)?

2. **Algorithmic Complexity**:
   - ReDoS vulnerable regex patterns?
   - XML bomb / billion laughs attack?
   - Zip bombs?
   - Hash collision attacks (HashDoS)?
   - Deeply nested JSON/XML?

3. **Database**:
   - Query timeout limits?
   - Connection pool limits?
   - Long-running query protection?
   - Full table scan triggers?

4. **Memory**:
   - Large payload handling?
   - Image/video processing limits?
   - Streaming vs loading entire files?

5. **External Dependencies**:
   - Timeout on all external calls?
   - Circuit breakers?
   - Fallback mechanisms?

For each DoS vector:
- The vulnerable endpoint/feature
- How to trigger resource exhaustion
- Resource limits to implement
```

---

## PHASE 12: COMPLIANCE & MONITORING

### Prompt 12.1: Logging & Monitoring Audit

```
Audit security logging and monitoring:

1. **What SHOULD Be Logged**:
   - Authentication success/failure
   - Authorization failures
   - Input validation failures
   - Application errors
   - Admin actions
   - Data access (sensitive data)
   - Configuration changes

2. **What SHOULD NOT Be Logged**:
   - Passwords (even failed attempts)
   - Full credit card numbers
   - Session tokens
   - API keys
   - PII (or must be masked)

3. **Log Quality**:
   - Timestamp (ISO 8601, UTC)?
   - User identifier?
   - IP address?
   - Action performed?
   - Success/failure?
   - Correlation ID for tracing?

4. **Log Security**:
   - Log injection possible?
   - Logs transmitted securely?
   - Log access restricted?
   - Log retention policy?

5. **Alerting**:
   - Brute force detection?
   - Anomaly detection?
   - Error rate spikes?
   - Suspicious patterns?

For gaps found:
- What's missing
- What should be added
- Implementation guidance
```

### Prompt 12.2: Security Checklist Verification

```
Final verification against security checklist:

## CRITICAL (Must fix before production)
[ ] No SQL injection vulnerabilities
[ ] No XSS vulnerabilities  
[ ] No hardcoded secrets
[ ] Passwords hashed with bcrypt/argon2
[ ] Authentication on all sensitive endpoints
[ ] Authorization checked server-side (no IDOR)
[ ] HTTPS only
[ ] CSRF protection on state-changing endpoints
[ ] No sensitive data in logs

## HIGH (Fix within 1 week)
[ ] Rate limiting on auth endpoints
[ ] Rate limiting on expensive operations
[ ] Security headers configured
[ ] Dependencies updated
[ ] File upload validation
[ ] Session management secure
[ ] Error messages don't leak info
[ ] CORS properly configured

## MEDIUM (Fix within 1 month)
[ ] Rate limiting comprehensive
[ ] Input validation comprehensive
[ ] Logging comprehensive
[ ] Account lockout implemented
[ ] Password reset secure
[ ] Session timeout implemented

## LOW (Continuous improvement)
[ ] Monitoring and alerting
[ ] Penetration testing scheduled
[ ] Security training for team
[ ] Incident response plan
[ ] Regular dependency updates

For each unchecked item:
- Current state
- What's needed
- Priority justification
```

---

# 📋 QUICK AUDIT COMMANDS

Paste these one-liners to quickly scan code:

```bash
# Find hardcoded secrets
grep -rn "password\|secret\|api_key\|apikey\|token\|private_key" --include="*.js" --include="*.ts" --include="*.py" --include="*.env*"

# Find SQL queries (potential injection)
grep -rn "SELECT\|INSERT\|UPDATE\|DELETE\|query(" --include="*.js" --include="*.ts" --include="*.py"

# Find dangerous functions
grep -rn "eval\|exec\|innerHTML\|dangerouslySetInnerHTML\|document.write" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx"

# Find TODO/FIXME security notes
grep -rn "TODO\|FIXME\|HACK\|XXX\|security\|vulnerable" --include="*.js" --include="*.ts" --include="*.py"

# Find console.log (shouldn't be in production)
grep -rn "console.log\|console.error\|print(" --include="*.js" --include="*.ts" --include="*.py"
```

---

# 🔄 CONTINUOUS SECURITY PROMPTS

## For Code Review
```
Review this code change for security issues:
1. Does it introduce new attack surface?
2. Does it handle user input safely?
3. Does it expose sensitive data?
4. Does it maintain authentication/authorization?
5. Does it have proper error handling?
```

## For New Features
```
Security review for new feature:
1. What data does it access?
2. Who should have access?
3. What can go wrong?
4. How can it be abused?
5. What should be logged?
```

## For Incident Response
```
Security incident analysis:
1. What was the attack vector?
2. What data was potentially exposed?
3. What is the blast radius?
4. What evidence do we have in logs?
5. What immediate fixes are needed?
6. What long-term fixes are needed?
```

---

# 📊 SEVERITY CLASSIFICATION

Use this for consistent severity ratings:

**CRITICAL** (Fix immediately, block deployment):
- Remote code execution
- SQL injection
- Authentication bypass
- Hardcoded production credentials
- Direct database exposure

**HIGH** (Fix within 24-48 hours):
- XSS (stored)
- IDOR with sensitive data
- CSRF on critical functions
- Weak password hashing
- Missing authentication

**MEDIUM** (Fix within 1 week):
- XSS (reflected)
- Missing rate limiting
- Verbose error messages
- Missing security headers
- Weak session management

**LOW** (Fix within 1 month):
- Information disclosure (minor)
- Missing logging
- Suboptimal configuration
- Minor best practice violations

---

Made with 🔒 for secure vibe coding

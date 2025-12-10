# UCC Student Portal - Security Checklist

**Version**: 1.0 MVP  
**Last Updated**: December 11, 2025  
**Status**: 🟢 APPROVED FOR DEVELOPMENT

---

## 📌 Overview

This document provides a comprehensive security checklist covering OWASP Top 10, data protection, privacy compliance, and deployment security for the UCC Student Portal. Every item must be verified before launching to production.

---

## 🔒 OWASP Top 10 (2023)

### **1. Broken Access Control**

**Risk**: Unauthorized users accessing other users' data or admin features.

**Preventions**:

```javascript
// ✅ Verify user ownership on every sensitive operation
async function deletePost(postId, userId) {
  const post = await db.prepare(
    'SELECT user_id FROM posts WHERE id = ?'
  ).bind(postId).first();
  
  if (post.user_id !== userId) {
    return error('FORBIDDEN', 'Not authorized to delete this post');
  }
  
  // Proceed with deletion
}

// ✅ Verify user can view resource
async function getUser(userId, requestingUserId) {
  const user = await db.prepare(
    'SELECT * FROM users WHERE id = ?'
  ).bind(userId).first();
  
  // Check if blocked
  const isBlocked = await db.prepare(
    'SELECT 1 FROM blocks WHERE blocker_id = ? AND blocked_id = ?'
  ).bind(user.id, requestingUserId).first();
  
  if (isBlocked) {
    return error('FORBIDDEN', 'User blocked you');
  }
  
  return user;
}

// ✅ Role-based access (future: admin features)
async function requireRole(request, requiredRole) {
  const user = request.user;
  
  if (user.role !== requiredRole && user.role !== 'admin') {
    return error('FORBIDDEN', `Requires ${requiredRole} role`);
  }
}
```

**Testing**:
- [ ] Try to edit another user's post (403 Forbidden)
- [ ] Try to delete another user's account (403 Forbidden)
- [ ] Try to access admin endpoints without admin role (403 Forbidden)
- [ ] Try to view blocked user's profile (403 Forbidden)

**Acceptance**: All unauthorized access attempts return 403 Forbidden

---

### **2. Cryptographic Failures**

**Risk**: Sensitive data exposed in transit or at rest.

**Preventions**:

```javascript
// ✅ HTTPS/TLS Enforcement
function enforceHTTPS(request) {
  if (!request.url.startsWith('https://')) {
    return error('INSECURE', 'HTTPS required');
  }
}

// ✅ Password Hashing (Bcrypt)
async function hashPassword(password) {
  const salt = await bcrypt.genSalt(12);
  return await bcrypt.hash(password, salt);
}

// ✅ Encrypt Sensitive Data
function encryptSensitiveData(data) {
  return crypto.encrypt(data, env.ENCRYPTION_KEY);
}

// ✅ Secure Token Generation
function generateSecureToken() {
  return crypto.randomBytes(32).toString('hex');
}

// ✅ HSTS Header (Force HTTPS)
headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains';

// ✅ No sensitive data in URLs
// ❌ DON'T: /reset-password?token=abc123&email=user@test.com
// ✅ DO: /reset-password?token=abc123 (email in JWT)
```

**Testing**:
- [ ] All API endpoints respond with HTTPS only
- [ ] Passwords stored as Bcrypt hashes, not plain text
- [ ] HSTS header present on all responses
- [ ] No sensitive data logged to console
- [ ] No sensitive data in error messages

**Acceptance**: Zero plain-text sensitive data in transit or at rest

---

### **3. Injection (SQL, Command, XSS)**

**Risk**: Attackers injecting malicious code into queries or HTML.

**Preventions**:

**SQL Injection Prevention**:
```javascript
// ✅ Use parameterized queries (Cloudflare D1)
const result = await db.prepare(
  'SELECT * FROM users WHERE email = ? AND is_active = ?'
).bind(email, true).first();

// ❌ DON'T: String concatenation
// const result = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ Validate input type
function validateEmail(email) {
  if (typeof email !== 'string') {
    throw new Error('Email must be string');
  }
  
  if (!/^[\w.-]+@ucc\.edu\.ph$/.test(email)) {
    throw new Error('Invalid email format');
  }
}
```

**XSS Prevention**:
```javascript
// ✅ Sanitize user input
import DOMPurify from 'isomorphic-dompurify';

function sanitizeUserContent(content) {
  return DOMPurify.sanitize(content, {
    ALLOWED_TAGS: [],  // No HTML tags allowed
    ALLOWED_ATTR: []
  });
}

// ✅ Escape output
function escapeHTML(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}

// ✅ Content Security Policy
headers['Content-Security-Policy'] = 
  "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'";

// ✅ No innerHTML with user data
// ❌ DON'T: element.innerHTML = userContent;
// ✅ DO: element.textContent = userContent;
```

**Command Injection Prevention**:
```javascript
// ✅ Never execute user input as commands
// ✅ Use APIs instead of shell commands
// ✅ Validate all file names

function validateFileName(filename) {
  // Whitelist allowed characters
  if (!/^[a-zA-Z0-9._-]+$/.test(filename)) {
    throw new Error('Invalid filename');
  }
  
  // Prevent path traversal
  if (filename.includes('..') || filename.includes('/')) {
    throw new Error('Invalid path');
  }
}
```

**Testing**:
- [ ] Try SQL injection: `' OR '1'='1`
- [ ] Try XSS: `<script>alert('xss')</script>`
- [ ] Try HTML injection: `<img src=x onerror=alert('xss')>`
- [ ] Try command injection in file names
- [ ] All injection attempts sanitized/rejected

**Acceptance**: Zero successful injection attacks

---

### **4. Insecure Design**

**Risk**: Application logic flaws exploitable by attackers.

**Preventions**:

```javascript
// ✅ Principle of Least Privilege
// Users can only access their own data by default
// Admin features explicitly restricted

// ✅ Rate Limiting (prevent abuse)
async function checkRateLimit(userId, action) {
  const key = `ratelimit:${userId}:${action}`;
  const current = await KV.get(key) || 0;
  
  const limits = {
    'create_post': { limit: 100, window: 86400 },
    'create_comment': { limit: 500, window: 86400 },
    'reaction': { limit: 1000, window: 86400 },
    'login_attempt': { limit: 10, window: 3600 }
  };
  
  if (current >= limits[action].limit) {
    return { allowed: false, retryAfter: limits[action].window };
  }
  
  await KV.put(key, current + 1, { 
    expirationTtl: limits[action].window 
  });
  
  return { allowed: true };
}

// ✅ Account Lockout (prevent brute force)
async function handleFailedLogin(email) {
  const user = await getUser(email);
  const newAttempts = user.login_attempts + 1;
  
  if (newAttempts >= 5) {
    const lockUntil = new Date(Date.now() + 15 * 60000);
    await updateUser(user.id, {
      login_locked_until: lockUntil
    });
  }
}

// ✅ Email Verification (prevent spam accounts)
// Account unusable until email verified

// ✅ Transaction Safety
async function createPost(userId, content) {
  try {
    await db.batch([
      db.prepare('INSERT INTO posts ...'),
      db.prepare('UPDATE users SET posts_count = posts_count + 1 ...'),
      db.prepare('INSERT INTO notifications ...')
    ]);
  } catch (err) {
    // All changes rolled back if any fail
    throw err;
  }
}
```

**Testing**:
- [ ] Rate limiting enforced on all endpoints
- [ ] Account lockout after 5 failed login attempts
- [ ] Email verification required before access
- [ ] Transaction consistency verified
- [ ] Idempotency for critical operations

**Acceptance**: All design flaws mitigated

---

### **5. Broken Authentication**

**Risk**: Users impersonating other accounts.

**Preventions**:

```javascript
// ✅ Strong Password Requirements
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;

// ✅ Secure Token Management
// - Access tokens: 15 min expiry
// - Refresh tokens: 7 day expiry with rotation
// - Tokens stored in HttpOnly cookies
// - Tokens signed and verified

// ✅ Multi-factor auth (future)
// - Email verification on new IP
// - TOTP support

// ✅ Session Management
async function createSession(userId) {
  const sessionId = crypto.randomUUID();
  const sessionData = {
    userId,
    ipAddress: getClientIP(),
    userAgent: getUserAgent(),
    createdAt: new Date().toISOString()
  };
  
  await SESSION_KV.put(
    `session:${sessionId}`,
    JSON.stringify(sessionData),
    { expirationTtl: 604800 }  // 7 days
  );
  
  return sessionId;
}

// ✅ IP Change Detection
function detectIPChange(currentIP, sessionIP) {
  if (currentIP !== sessionIP) {
    logSecurityEvent('IP_CHANGE', { currentIP, sessionIP });
    // Optionally: Require re-authentication
  }
}
```

**Testing**:
- [ ] Cannot bypass email verification
- [ ] Cannot use expired tokens
- [ ] Cannot forge valid tokens
- [ ] Session invalidated on logout
- [ ] Password reset tokens expire after 1 hour
- [ ] Failed login attempts tracked

**Acceptance**: No authentication bypass vulnerabilities

---

### **6. Sensitive Data Exposure**

**Risk**: PII exposed in logs, errors, or API responses.

**Preventions**:

```javascript
// ✅ Never log sensitive data
// ❌ DON'T: console.log({ email, password, token });
// ✅ DO: console.log({ userId, action: 'login_success' });

// ✅ Mask sensitive fields in responses
function maskResponse(data) {
  return {
    ...data,
    email: maskEmail(data.email),
    phone: maskPhone(data.phone)
  };
}

function maskEmail(email) {
  const [name, domain] = email.split('@');
  const masked = name.slice(0, 1) + '*'.repeat(name.length - 1) + '@' + domain;
  return masked;  // j***@ucc.edu.ph
}

// ✅ PII storage minimization
// - Only collect necessary data
// - Delete old logs (30 day retention)
// - Anonymize audit logs after 1 year

// ✅ Error message sanitization
// ❌ DON'T: "User with email foo@bar.com not found"
// ✅ DO: "Invalid email or password" (generic)

function sanitizeError(error) {
  // Don't expose database errors to users
  if (error.code === 'SQLITE_READONLY') {
    return 'Database error. Please try again later.';
  }
  
  return error.message;
}
```

**Testing**:
- [ ] No passwords in logs or error messages
- [ ] No API tokens in responses
- [ ] Error messages generic (no database details)
- [ ] PII never logged to console
- [ ] Audit logs don't contain sensitive data
- [ ] Old logs deleted per retention policy

**Acceptance**: Zero sensitive data exposure

---

### **7. Identification & Authentication Failures**

**Risk**: Users' identities compromised.

**Preventions**:

```javascript
// ✅ Unique email verification
// Each email can only be used once

// ✅ Account recovery (email only)
// - Verification code sent to registered email
// - Code expires after 1 hour
// - Code single-use

// ✅ Session timeout
const SESSION_TIMEOUT = 30 * 60 * 1000;  // 30 minutes inactivity

async function validateSession(sessionId) {
  const session = await SESSION_KV.get(`session:${sessionId}`);
  
  if (!session) {
    return { valid: false };
  }
  
  const data = JSON.parse(session);
  const lastActive = new Date(data.lastActive);
  const inactiveTime = Date.now() - lastActive;
  
  if (inactiveTime > SESSION_TIMEOUT) {
    await SESSION_KV.delete(`session:${sessionId}`);
    return { valid: false, reason: 'session_expired' };
  }
  
  // Update last activity
  data.lastActive = new Date().toISOString();
  await SESSION_KV.put(`session:${sessionId}`, JSON.stringify(data), {
    expirationTtl: 604800
  });
  
  return { valid: true };
}

// ✅ Password reset security
// - Token sent to email only (not SMS)
// - Token single-use
// - Token has expiration
// - Old passwords invalidated
```

**Testing**:
- [ ] Each email can only register once
- [ ] Session timeout enforced (30 min inactivity)
- [ ] Password reset tokens single-use
- [ ] Account recovery requires email access
- [ ] Old sessions invalidated on logout

**Acceptance**: All authentication mechanisms secure

---

### **8. Software & Data Integrity Failures**

**Risk**: Malicious code or data modification.

**Preventions**:

```javascript
// ✅ Dependency scanning
// - npm audit run before each deployment
// - Known vulnerabilities auto-detected
// - Critical vulnerabilities block deployment

// ✅ Integrity checks
// - File checksums verified
// - Responses signed with HMAC
// - Database transactions atomic

// ✅ Secure CI/CD
// - Only authorized users can deploy
// - All deployments logged
// - Automatic rollback on errors
// - Code review required before merge

// ✅ Subresource Integrity (SRI) for CDN
// <script src="https://cdn.example.com/lib.js" 
//   integrity="sha384-..."></script>

// ✅ Webhook verification
async function handleWebhook(request) {
  const signature = request.headers.get('x-signature');
  const body = await request.text();
  
  // Verify HMAC signature
  const computed = await computeHMAC(body, env.WEBHOOK_SECRET);
  
  if (signature !== computed) {
    return error('INVALID_SIGNATURE', 'Webhook signature invalid');
  }
}
```

**Testing**:
- [ ] npm audit passes (no high/critical vulns)
- [ ] Dependencies updated regularly
- [ ] Code review required before deploy
- [ ] Automated tests pass before deploy
- [ ] Webhook signatures verified

**Acceptance**: All deployments safe and verified

---

### **9. Logging & Monitoring Failures**

**Risk**: Security breaches go unnoticed.

**Preventions**:

```javascript
// ✅ Comprehensive logging
async function logSecurityEvent(event, data) {
  await LOGS_KV.put(
    `log:${Date.now()}:${crypto.randomUUID()}`,
    JSON.stringify({
      event,
      timestamp: new Date().toISOString(),
      userId: data.userId,
      ipAddress: data.ipAddress,
      userAgent: data.userAgent,
      details: data.details,
      severity: data.severity || 'info'
    }),
    { expirationTtl: 30 * 86400 }  // 30 day retention
  );
}

// Critical events to log:
const logEvents = [
  'user_registration',
  'user_login',
  'failed_login_attempt',
  'account_locked',
  'email_verification',
  'password_reset',
  'suspicious_activity',
  'rate_limit_exceeded',
  'unauthorized_access',
  'data_deletion',
  'admin_action'
];

// ✅ Alerts on suspicious activity
async function setupAlerts() {
  // Alert if:
  // - 5+ failed login attempts from one IP
  // - 50+ requests from one IP in 1 minute
  // - Unusual geographic access
  // - Multiple accounts from same IP in 1 hour
  // - Post with many flags
  // - Comments with many flags
}

// ✅ Regular log review
// - Daily security log review
// - Weekly analytics
// - Monthly incident analysis
```

**Testing**:
- [ ] All critical events logged
- [ ] Logs include timestamp, user, action
- [ ] Logs retained for 30 days
- [ ] Alerts configured for anomalies
- [ ] Log access restricted to admins
- [ ] Logs not exposed in APIs

**Acceptance**: Comprehensive logging and alerting

---

### **10. SSRF (Server-Side Request Forgery)**

**Risk**: Attackers making requests to internal resources.

**Preventions**:

```javascript
// ✅ Whitelist allowed URLs
async function downloadFile(url) {
  // Only allow R2 URLs
  if (!url.startsWith('https://r2.example.com/')) {
    throw new Error('Invalid file URL');
  }
  
  return await fetch(url);
}

// ✅ Block internal IP ranges
function isInternalIP(ip) {
  const internalRanges = [
    /^127\./,           // Localhost
    /^10\./,            // Private
    /^172\.(1[6-9]|2[0-9]|3[0-1])\./,  // Private
    /^192\.168\./,      // Private
    /^::1$/,            // IPv6 localhost
    /^fc00:/            // IPv6 private
  ];
  
  return internalRanges.some(range => range.test(ip));
}

// ✅ Validate redirects
async function followRedirect(url, maxRedirects = 5) {
  let currentUrl = url;
  let redirects = 0;
  
  while (redirects < maxRedirects) {
    const response = await fetch(currentUrl, { redirect: 'manual' });
    
    if (response.status < 300 || response.status >= 400) {
      return response;
    }
    
    const nextUrl = response.headers.get('location');
    
    // Validate redirect target
    if (!isValidRedirectTarget(nextUrl)) {
      throw new Error('Suspicious redirect');
    }
    
    currentUrl = nextUrl;
    redirects++;
  }
}
```

**Testing**:
- [ ] Cannot access internal IPs (127.0.0.1, 192.168.x.x)
- [ ] Cannot access metadata services (169.254.169.254)
- [ ] Only whitelisted domains allowed
- [ ] Redirect chains validated
- [ ] DNS rebinding prevented

**Acceptance**: SSRF vulnerabilities mitigated

---

## 🛡️ Additional Security Requirements

### **Data Encryption**

```javascript
// ✅ Passwords: Bcrypt (salted)
// ✅ Tokens: JWT (signed, not encrypted - ok for non-PII)
// ✅ Sensitive data at rest: AES-256-GCM
// ✅ Data in transit: TLS 1.3

const encryptionAlgorithm = 'aes-256-gcm';
const encryptionKeyLength = 32;  // 256 bits

async function encryptData(plaintext, key) {
  const iv = crypto.randomBytes(12);
  const cipher = crypto.createCipheriv(encryptionAlgorithm, key, iv);
  
  const encrypted = Buffer.concat([
    cipher.update(plaintext, 'utf8'),
    cipher.final()
  ]);
  
  const authTag = cipher.getAuthTag();
  
  return {
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex'),
    encrypted: encrypted.toString('hex')
  };
}
```

### **Privacy Compliance**

**GDPR Compliance**:
```
✅ User consent for data collection
✅ Data minimization (collect only necessary data)
✅ User right to access their data
✅ User right to delete their data
✅ User right to export their data (GDPR portability)
✅ Breach notification within 72 hours
✅ Data processing agreement with subprocessors
✅ Privacy policy publicly available
```

**CCPA Compliance** (if US-based students):
```
✅ Privacy notice at collection
✅ Right to know what data is collected
✅ Right to delete personal information
✅ Right to opt-out of sale (if applicable)
✅ No discrimination for exercising rights
```

### **Student Privacy Protections**

```javascript
// ✅ Block minors (must be 18+)
function validateAge(dateOfBirth) {
  const age = new Date().getFullYear() - dateOfBirth.getFullYear();
  
  if (age < 18) {
    throw new Error('Must be 18+ to use platform');
  }
}

// ✅ Parental consent for minors (if applicable)
// ✅ No algorithm-based targeting on students
// ✅ No data sharing with third parties without consent
// ✅ No behavioral profiling
// ✅ No targeted advertising
```

---

## 🔐 Deployment Security

### **Before Launch Checklist**

- [ ] All dependencies up to date (`npm audit`)
- [ ] No hardcoded secrets (all in environment variables)
- [ ] HTTPS/TLS enabled
- [ ] HSTS header configured
- [ ] CSP header configured
- [ ] Cloudflare firewall rules enabled
- [ ] Rate limiting configured
- [ ] DDoS protection enabled (Cloudflare)
- [ ] Backup strategy configured
- [ ] Disaster recovery plan documented
- [ ] Security headers all set
- [ ] CORS properly configured
- [ ] Admin password strong and rotated
- [ ] Database backups scheduled
- [ ] Monitoring and alerting configured
- [ ] Incident response plan documented
- [ ] Security documentation completed
- [ ] Penetration testing completed (optional)
- [ ] All known vulnerabilities addressed
- [ ] Security team briefed on deployment

### **Environment Variables (Never in Code)**

```
JWT_SECRET=<random-64-char-string>
ENCRYPTION_KEY=<random-32-byte-hex>
DATABASE_URL=<cloudflare-d1-url>
R2_BUCKET_NAME=ucc-student-portal-files
R2_ACCESS_KEY_ID=<from-cloudflare>
R2_SECRET_ACCESS_KEY=<from-cloudflare>
WEBHOOK_SECRET=<random-32-char>
SENDGRID_API_KEY=<from-sendgrid>
SENTRY_DSN=<for-error-tracking>
ENVIRONMENT=production
```

### **Network Security**

```
✅ Cloudflare DDoS protection enabled
✅ Cloudflare WAF rules configured
✅ Rate limiting per IP and user
✅ Geo-blocking (if needed)
✅ Bot detection enabled
✅ CAPTCHA on login after failed attempts
✅ Firewall rules restrict admin endpoints
✅ VPN/Proxy detection enabled (optional)
```

---

## 📊 Security Testing Checklist

### **Manual Testing**

- [ ] Try SQL injection in login form
- [ ] Try XSS in post content
- [ ] Try CSRF attack
- [ ] Try accessing other user's profile
- [ ] Try editing other user's posts
- [ ] Try brute forcing password
- [ ] Try expired token
- [ ] Try forged token
- [ ] Try SSRF with internal URLs

### **Automated Testing**

- [ ] OWASP ZAP scanning
- [ ] Dependency vulnerability scan
- [ ] SonarQube code quality
- [ ] SAST (Static Application Security Testing)
- [ ] DAST (Dynamic Application Security Testing)

### **Third-Party Tools**

- [ ] npm audit (dependencies)
- [ ] Snyk (vulnerability management)
- [ ] Checkmarx (SAST)
- [ ] Burp Suite (penetration testing)
- [ ] Nessus (vulnerability scanning)

---

## ✅ Final Security Approval Checklist

**Before going live, verify ALL items:**

- [ ] OWASP Top 10 covered
- [ ] No hardcoded secrets
- [ ] HTTPS enforced
- [ ] Passwords properly hashed
- [ ] Tokens properly signed
- [ ] XSS protection enabled
- [ ] CSRF protection enabled
- [ ] SQL injection prevented
- [ ] Rate limiting working
- [ ] Account lockout working
- [ ] Email verification required
- [ ] Logging comprehensive
- [ ] Monitoring alerts configured
- [ ] Backup strategy in place
- [ ] Disaster recovery plan documented
- [ ] All tests passing
- [ ] Security dependencies updated
- [ ] Penetration testing completed
- [ ] Team trained on security
- [ ] Incident response plan ready

**Security Sign-off**: _____________  Date: ___________

---

**Documentation Complete ✅**

All 5 critical technical documents have been created and are production-ready.

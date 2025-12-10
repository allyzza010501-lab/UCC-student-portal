# UCC Student Portal - Authentication Guide

**Version**: 1.0 MVP  
**Last Updated**: December 11, 2025  
**Status**: 🟢 APPROVED FOR DEVELOPMENT

---

## 📌 Overview

This document defines the complete authentication and authorization system for the UCC Student Portal. The system ensures only verified UCC students can access the platform while protecting user accounts with industry-standard security.

---

## 🎯 Authentication Strategy

### **Core Principles**

1. **Student-Only Access**: Email verification required (`@ucc.edu.ph` domain)
2. **Zero-Trust Architecture**: Every request authenticated, no implicit trust
3. **Stateless Tokens**: JWT-based (no server-side sessions needed)
4. **Rate Limiting**: Prevent brute force and abuse attacks
5. **Account Security**: Password hashing, account lockout, session management
6. **Compliance**: OWASP Top 10, GDPR-ready, student privacy protected

### **Three-Factor Security**

```
Factor 1: Email Verification
  ↓
Factor 2: Password Authentication
  ↓
Factor 3: Token-Based Access Control
```

---

## 📧 Email Verification Workflow

### **Why Email Verification is Critical**

- ✅ Ensures only real UCC students can register
- ✅ Prevents spam accounts and impersonation
- ✅ Validates student email is active
- ✅ Creates audit trail of account creation

### **Verification Process - Flow Diagram**

```
1. User enters email (must @ucc.edu.ph)
   │
   ├─→ Server validates email format
   │    └─→ Reject if not UCC domain
   │
2. User enters password (validated)
   │
   ├─→ Server generates verification token
   │    └─→ Token expires in 1 hour
   │
3. Server sends email with verification link
   │
   ├─→ "Click here to verify: 
   │    heralds.pages.dev/verify?token=abc123"
   │
4. User clicks link in email
   │
   ├─→ Frontend submits token to /verify-email
   │    └─→ Token must be valid and not expired
   │
5. Server validates token
   │
   ├─→ If valid: Mark email_verified = true
   │    └─→ Issue JWT access token
   │
6. User is logged in and redirected to profile setup
```

### **Verification Token Specifications**

```javascript
{
  type: "email_verification",
  user_id: "uuid-123",
  email: "student@ucc.edu.ph",
  issued_at: "2025-12-11T10:00:00Z",
  expires_at: "2025-12-11T11:00:00Z",
  nonce: "random-secure-value"
}
```

**Token Generation Code** (Pseudocode):
```javascript
function generateVerificationToken(user) {
  const token = {
    type: "email_verification",
    user_id: user.id,
    email: user.email,
    issued_at: new Date().toISOString(),
    expires_at: new Date(Date.now() + 3600000).toISOString(), // 1 hour
    nonce: crypto.randomBytes(32).toString('hex')
  };
  
  // Sign with secret key
  const signed = jwt.sign(token, env.JWT_SECRET, {
    algorithm: 'HS256',
    expiresIn: '1h'
  });
  
  return signed;
}
```

---

## 🔐 Password Security

### **Password Requirements**

```
✅ Minimum 8 characters
✅ At least 1 uppercase letter (A-Z)
✅ At least 1 lowercase letter (a-z)
✅ At least 1 number (0-9)
✅ At least 1 special character (!@#$%^&*)
❌ Not common passwords (checked against dictionary)
❌ Not user's email or name
```

**Validation Regex**:
```javascript
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
```

### **Password Hashing**

```javascript
import bcrypt from 'bcrypt';

// During registration
async function hashPassword(password) {
  const salt = await bcrypt.genSalt(12); // 12 rounds
  return await bcrypt.hash(password, salt);
}

// During login
async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}
```

**Why Bcrypt?**
- ✅ Automatically salts password
- ✅ Slows down brute force attacks
- ✅ 12 rounds = ~250ms per hash (security sweet spot)
- ✅ Industry standard for password hashing

---

## 🎫 JWT Token System

### **Access Token (Short-lived)**

```javascript
{
  "sub": "uuid-123",                    // Subject (user_id)
  "email": "student@ucc.edu.ph",
  "iat": 1702336800,                    // Issued at
  "exp": 1702337700,                    // Expires at (15 min later)
  "type": "access",
  "scope": ["posts:read", "posts:write", "profile:read"]
}
```

**Duration**: 15 minutes
**Refresh**: Via refresh token
**Storage**: HttpOnly secure cookie (cannot be accessed by JavaScript)

### **Refresh Token (Long-lived)**

```javascript
{
  "sub": "uuid-123",
  "email": "student@ucc.edu.ph",
  "iat": 1702336800,
  "exp": 1702941600,                    // Expires at (7 days later)
  "type": "refresh",
  "nonce": "unique-per-token"
}
```

**Duration**: 7 days
**Use Case**: Getting new access tokens without re-entering password
**Storage**: HttpOnly secure cookie
**Rotation**: New refresh token issued on each refresh

### **Token Signing & Verification**

```javascript
// Signing
const token = jwt.sign(payload, env.JWT_SECRET, {
  algorithm: 'HS256',
  expiresIn: '15m'  // for access tokens
});

// Verification (on every request)
try {
  const decoded = jwt.verify(token, env.JWT_SECRET, {
    algorithms: ['HS256']
  });
  // Token valid, use decoded payload
} catch (err) {
  // Token invalid, expired, or tampered with
  // Return 401 Unauthorized
}
```

### **Token Middleware** (Every Protected Endpoint)

```javascript
async function authenticateRequest(request) {
  // Extract token from Authorization header or cookies
  const token = extractTokenFromRequest(request);
  
  if (!token) {
    return new Response(JSON.stringify({
      success: false,
      error: {
        code: 'NO_TOKEN',
        message: 'Authorization token required'
      }
    }), { status: 401 });
  }
  
  try {
    const decoded = jwt.verify(token, env.JWT_SECRET);
    
    // Attach user to request context
    request.user = {
      id: decoded.sub,
      email: decoded.email,
      scopes: decoded.scope
    };
    
    return null; // No error, proceed
  } catch (err) {
    return new Response(JSON.stringify({
      success: false,
      error: {
        code: 'INVALID_TOKEN',
        message: 'Token is invalid or expired'
      }
    }), { status: 401 });
  }
}
```

---

## 🔄 Complete Login Flow

### **Step 1: User Submits Email & Password**

```
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "student@ucc.edu.ph",
  "password": "SecurePass123!"
}
```

### **Step 2: Server Validates Input**

```javascript
async function handleLogin(request) {
  const { email, password } = await request.json();
  
  // Validate email format and UCC domain
  if (!email.endsWith('@ucc.edu.ph') && !email.endsWith('@students.ucc.edu.ph')) {
    return error('INVALID_EMAIL', 'Email must be @ucc.edu.ph');
  }
  
  // Validate password format
  if (!isValidPassword(password)) {
    return error('WEAK_PASSWORD', 'Password does not meet requirements');
  }
  
  // Continue to next step
}
```

### **Step 3: Server Checks for Account**

```javascript
// Query database
const user = await db.prepare(
  'SELECT * FROM users WHERE email = ? AND is_active = true'
).bind(email).first();

if (!user) {
  // Log attempt for security audit
  logLoginAttempt(email, 'USER_NOT_FOUND');
  
  // Return generic message (don't reveal if email exists)
  return error('INVALID_CREDENTIALS', 'Email or password is incorrect');
}
```

### **Step 4: Check Email Verification**

```javascript
if (!user.email_verified) {
  return error('EMAIL_NOT_VERIFIED', 
    'Please verify your email before logging in',
    { resend_verification: true }
  );
}
```

### **Step 5: Check Account Lockout**

```javascript
if (user.login_locked_until) {
  const lockoutExpired = new Date() > new Date(user.login_locked_until);
  
  if (!lockoutExpired) {
    const remainingTime = Math.ceil(
      (new Date(user.login_locked_until) - new Date()) / 1000
    );
    return error('ACCOUNT_LOCKED',
      `Account locked. Try again in ${remainingTime} seconds`
    );
  }
  
  // Lockout expired, reset
  await resetLoginAttempts(user.id);
}
```

### **Step 6: Verify Password**

```javascript
const passwordValid = await bcrypt.compare(password, user.password_hash);

if (!passwordValid) {
  // Increment failed attempt counter
  const newAttempts = user.login_attempts + 1;
  
  if (newAttempts >= 5) {
    // Lock account for 15 minutes
    const lockUntil = new Date(Date.now() + 15 * 60000);
    await db.prepare(
      'UPDATE users SET login_attempts = ?, login_locked_until = ? WHERE id = ?'
    ).bind(newAttempts, lockUntil.toISOString(), user.id).run();
    
    return error('TOO_MANY_ATTEMPTS', 'Account locked due to failed attempts');
  }
  
  // Update attempt count
  await db.prepare(
    'UPDATE users SET login_attempts = ? WHERE id = ?'
  ).bind(newAttempts, user.id).run();
  
  return error('INVALID_CREDENTIALS', 'Email or password is incorrect');
}
```

### **Step 7: Reset Failed Attempts**

```javascript
await db.prepare(
  'UPDATE users SET login_attempts = 0, login_locked_until = NULL WHERE id = ?'
).bind(user.id).run();
```

### **Step 8: Generate Tokens**

```javascript
// Access token (15 min)
const accessToken = jwt.sign({
  sub: user.id,
  email: user.email,
  type: 'access',
  scope: ['posts:read', 'posts:write', 'profile:read', 'profile:write']
}, env.JWT_SECRET, {
  algorithm: 'HS256',
  expiresIn: '15m'
});

// Refresh token (7 days)
const refreshToken = jwt.sign({
  sub: user.id,
  email: user.email,
  type: 'refresh',
  nonce: crypto.randomBytes(16).toString('hex')
}, env.JWT_SECRET, {
  algorithm: 'HS256',
  expiresIn: '7d'
});
```

### **Step 9: Update Last Login**

```javascript
await db.prepare(
  'UPDATE users SET last_login_at = ? WHERE id = ?'
).bind(new Date().toISOString(), user.id).run();
```

### **Step 10: Return Tokens to Client**

```javascript
const response = {
  success: true,
  data: {
    user_id: user.id,
    email: user.email,
    first_name: user.first_name,
    access_token: accessToken,
    token_expires_in: 900,  // 15 min in seconds
    user: {
      id: user.id,
      email: user.email,
      profile_picture_url: user.profile_picture_url
    }
  }
};

// Set tokens in HttpOnly cookies
return new Response(JSON.stringify(response), {
  status: 200,
  headers: {
    'Set-Cookie': [
      `access_token=${accessToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=900`,
      `refresh_token=${refreshToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800`
    ]
  }
});
```

---

## 🔁 Token Refresh Flow

### **When Access Token Expires**

```
1. Frontend makes request with expired token
   ↓
2. Server responds with 401 "token expired"
   ↓
3. Frontend automatically calls POST /api/v1/auth/refresh-token
   ↓
4. Server checks refresh token (still valid?)
   ↓
5. Server issues new access token
   ↓
6. Frontend retries original request with new token
```

### **Refresh Token Implementation**

```javascript
async function handleRefreshToken(request) {
  // Extract refresh token from cookies
  const refreshToken = extractRefreshTokenFromCookie(request);
  
  if (!refreshToken) {
    return error('NO_REFRESH_TOKEN', 'Refresh token required');
  }
  
  try {
    // Verify refresh token
    const decoded = jwt.verify(refreshToken, env.JWT_SECRET);
    
    if (decoded.type !== 'refresh') {
      return error('INVALID_TOKEN_TYPE', 'Token must be refresh token');
    }
    
    // Get updated user info from database
    const user = await db.prepare(
      'SELECT id, email, is_active FROM users WHERE id = ?'
    ).bind(decoded.sub).first();
    
    if (!user || !user.is_active) {
      return error('USER_INACTIVE', 'User account is inactive');
    }
    
    // Issue new access token
    const newAccessToken = jwt.sign({
      sub: user.id,
      email: user.email,
      type: 'access',
      scope: ['posts:read', 'posts:write', 'profile:read', 'profile:write']
    }, env.JWT_SECRET, {
      algorithm: 'HS256',
      expiresIn: '15m'
    });
    
    // Also issue new refresh token (rotation)
    const newRefreshToken = jwt.sign({
      sub: user.id,
      email: user.email,
      type: 'refresh',
      nonce: crypto.randomBytes(16).toString('hex')
    }, env.JWT_SECRET, {
      algorithm: 'HS256',
      expiresIn: '7d'
    });
    
    return {
      success: true,
      data: {
        access_token: newAccessToken,
        token_expires_in: 900
      },
      headers: {
        'Set-Cookie': [
          `access_token=${newAccessToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=900`,
          `refresh_token=${newRefreshToken}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800`
        ]
      }
    };
  } catch (err) {
    return error('INVALID_REFRESH_TOKEN', 'Refresh token is invalid or expired');
  }
}
```

---

## 🚫 Security Best Practices

### **Rate Limiting on Auth Endpoints**

```javascript
// Per IP address rate limiting
const rateLimits = {
  '/auth/login': { attempts: 10, window: '1 hour' },
  '/auth/register': { attempts: 5, window: '1 hour' },
  '/auth/forgot-password': { attempts: 3, window: '1 hour' },
  '/auth/verify-email': { attempts: 5, window: '1 hour' }
};

// Implementation using Cloudflare KV for distributed rate limiting
async function checkRateLimit(ip, endpoint) {
  const key = `ratelimit:${endpoint}:${ip}`;
  const current = await RATE_LIMIT_KV.get(key) || 0;
  
  if (current >= rateLimits[endpoint].attempts) {
    return {
      allowed: false,
      retryAfter: '1 hour'
    };
  }
  
  await RATE_LIMIT_KV.put(key, current + 1, { expirationTtl: 3600 });
  return { allowed: true };
}
```

### **Session Management**

```javascript
// Session data stored in Cloudflare KV (distributed)
async function createSession(userId) {
  const sessionId = crypto.randomUUID();
  const sessionData = {
    userId,
    createdAt: new Date().toISOString(),
    lastActive: new Date().toISOString(),
    ipAddress: getClientIP(),
    userAgent: getUserAgent()
  };
  
  // Store in KV with 7-day expiration
  await SESSION_KV.put(
    `session:${sessionId}`,
    JSON.stringify(sessionData),
    { expirationTtl: 604800 }
  );
  
  return sessionId;
}

// Verify session on each request
async function verifySession(sessionId) {
  const session = await SESSION_KV.get(`session:${sessionId}`);
  
  if (!session) {
    return { valid: false, reason: 'Session not found' };
  }
  
  const data = JSON.parse(session);
  
  // Check if IP hasn't changed dramatically
  const currentIP = getClientIP();
  if (data.ipAddress !== currentIP) {
    // Optional: Require re-authentication
    logSecurityEvent('IP_CHANGE', data.userId);
  }
  
  // Update last active timestamp
  data.lastActive = new Date().toISOString();
  await SESSION_KV.put(`session:${sessionId}`, JSON.stringify(data), {
    expirationTtl: 604800
  });
  
  return { valid: true, data };
}
```

### **HTTPS/TLS Requirement**

```javascript
// Every auth endpoint must use HTTPS
function enforceHTTPS(request) {
  if (!request.url.startsWith('https://')) {
    return error('INSECURE_CONNECTION', 'HTTPS required for authentication');
  }
}
```

### **CSRF Protection**

```javascript
// CSRF token for login form
function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Verify on form submission
async function verifyCSRFToken(token) {
  const stored = await CSRF_KV.get(`csrf:${token}`);
  
  if (!stored) {
    return error('INVALID_CSRF', 'Security token is invalid or expired');
  }
  
  // Delete token after use (one-time use)
  await CSRF_KV.delete(`csrf:${token}`);
  return { valid: true };
}
```

---

## 🔑 Password Reset Flow

### **Step 1: User Requests Reset**

```
POST /api/v1/auth/forgot-password
{
  "email": "student@ucc.edu.ph"
}
```

### **Step 2: Server Validates User Exists**

```javascript
const user = await db.prepare(
  'SELECT id, email FROM users WHERE email = ?'
).bind(email).first();

// Important: Always return success (don't reveal if email exists)
if (!user) {
  return {
    success: true,
    data: {
      message: 'If email exists, password reset link has been sent'
    }
  };
}
```

### **Step 3: Generate Reset Token**

```javascript
const resetToken = jwt.sign({
  type: 'password_reset',
  user_id: user.id,
  email: user.email,
  nonce: crypto.randomBytes(16).toString('hex')
}, env.JWT_SECRET, {
  expiresIn: '1h'
});

// Store token in database for verification
await db.prepare(
  'UPDATE users SET reset_token = ?, reset_expires_at = ? WHERE id = ?'
).bind(resetToken, new Date(Date.now() + 3600000).toISOString(), user.id).run();
```

### **Step 4: Send Reset Email**

```
Subject: Reset Your Heralds Password
To: student@ucc.edu.ph

Click here to reset your password:
https://heralds.pages.dev/reset-password?token=abc123def456

This link expires in 1 hour.

If you didn't request this reset, ignore this email.
```

### **Step 5: User Clicks Link & Submits New Password**

```
POST /api/v1/auth/reset-password
{
  "token": "abc123def456",
  "password": "NewSecurePass123!"
}
```

### **Step 6: Server Validates & Updates Password**

```javascript
async function handleResetPassword(token, newPassword) {
  try {
    // Verify token
    const decoded = jwt.verify(token, env.JWT_SECRET);
    
    if (decoded.type !== 'password_reset') {
      return error('INVALID_TOKEN', 'Invalid reset token');
    }
    
    // Get user and check reset_expires_at
    const user = await db.prepare(
      'SELECT * FROM users WHERE id = ? AND reset_token = ?'
    ).bind(decoded.user_id, token).first();
    
    if (!user) {
      return error('TOKEN_NOT_FOUND', 'Reset token not found or expired');
    }
    
    if (new Date() > new Date(user.reset_expires_at)) {
      return error('TOKEN_EXPIRED', 'Reset token has expired');
    }
    
    // Hash new password
    const passwordHash = await bcrypt.hash(newPassword, 12);
    
    // Update password and clear reset token
    await db.prepare(
      'UPDATE users SET password_hash = ?, reset_token = NULL, reset_expires_at = NULL WHERE id = ?'
    ).bind(passwordHash, user.id).run();
    
    // Log password change for security audit
    logSecurityEvent('PASSWORD_RESET', user.id);
    
    return {
      success: true,
      data: {
        message: 'Password reset successful',
        redirect_to: '/login'
      }
    };
  } catch (err) {
    return error('RESET_FAILED', 'Password reset failed');
  }
}
```

---

## 📊 Authentication Security Checklist

- [ ] All passwords hashed with Bcrypt (12 rounds minimum)
- [ ] JWT tokens expire in 15 minutes (access) and 7 days (refresh)
- [ ] Tokens stored in HttpOnly, Secure, SameSite cookies
- [ ] Rate limiting on authentication endpoints (5-10 attempts/hour)
- [ ] Account lockout after 5 failed login attempts (15 minutes)
- [ ] Email verification required before account access
- [ ] UCC domain validation (@ucc.edu.ph) enforced
- [ ] HTTPS/TLS required on all auth endpoints
- [ ] CSRF tokens implemented on login form
- [ ] Session tracking with IP validation
- [ ] Password reset tokens expire after 1 hour
- [ ] Failed login attempts logged for audit
- [ ] Password complexity requirements enforced
- [ ] User agent/IP changes detected and logged
- [ ] Security headers set on all responses (HSTS, CSP, etc.)

---

**Next Document**: FEATURE_SPECIFICATIONS.md

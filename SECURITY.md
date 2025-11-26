# Cinco Coffee Security Implementation

## Overview

This document outlines the security solutions implemented to protect user data and prevent common web vulnerabilities.

---

## Solution 1: Password Encryption (Bcrypt Hashing)

### Bcrypt Password Hashing

We replaced plain-text password storage with bcrypt hashing, ensuring all passwords are cryptographically protected and unreadable even if an attacker accesses the browser's storage.

### Why Bcrypt Helps

Plain-text passwords are extremely dangerous because anyone with browser access could see every user's password. With bcrypt hashing, attackers only see irreversible scrambled strings like `$2b$10$KIXw...`, directly addressing the threat of stealing user passwords.

### Bcrypt Implementation Details

During account creation, bcrypt cryptographically hashes passwords with unique salts. At login, the entered password is hashed and compared to the stored hash for authentication.

### Bcrypt Configuration

- Bcrypt library: `bcrypt.min.js`
- Hash strength: 10 salt rounds
- Storage: Hashed passwords in localStorage
- Comparison: Constant-time comparison prevents timing attacks

---

## Solution 2: Input Validation and Sanitization

### Comprehensive Input Validation

We implemented comprehensive input validation and sanitization across all user-facing forms, checking for dangerous characters, limiting input length, validating data types, and stripping HTML/script tags to prevent malicious inputs.

### Why Input Validation Helps

The original system's lack of input validation made it vulnerable to XSS attacks where attackers could inject malicious JavaScript through forms. Our validation now detects and rejects dangerous patterns before they execute.

### Input Validation Implementation

User inputs pass through validation functions that:

- Check length limits
- Validate allowed characters
- Require fields
- Detect proper formats
- Strip script tags, SQL commands
- Remove suspicious characters

### Validation Functions

| Function             | Purpose             | Rules                                           |
| -------------------- | ------------------- | ----------------------------------------------- |
| `sanitizeInput()`    | Remove HTML/scripts | Strips `<script>`, `<iframe>`, dangerous tags   |
| `validateEmail()`    | Email validation    | Regex pattern + max 254 chars                   |
| `validatePassword()` | Password validation | 6-128 chars, alphanumeric + symbols             |
| `validateName()`     | Name validation     | 2-100 chars, letters/spaces/hyphens/apostrophes |
| `validatePhone()`    | Phone validation    | 10-15 digits only                               |

---

## Solution 3: Session Management with Auto-Timeout

### Automatic Session Timeout

We implemented automatic session timeout after 30 minutes of inactivity, tracking sessions with tokens that expire and require re-login.

### Why Session Timeout Helps

The original system's indefinite login sessions made accounts vulnerable to session hijacking. Timeout limits attackers to 30 minutes of access even if they steal credentials.

### Session Timeout Implementation

Upon login:

1. Session object created with email, unique token, and timestamp
2. Timestamp updates with each interaction
3. Background timer logs users out after 30 minutes of inactivity

### Session Configuration

```javascript
const SESSION_TIMEOUT_MINUTES = 30;
const SESSION_MONITOR_INTERVAL = 30000; // Check every 30 seconds
```

### Session Management Features

- Unique token per session (cryptographically generated)
- Automatic renewal on user activity
- Background monitoring every 30 seconds
- Secure logout clears all session data

---

## Solution 4: Rate Limiting and CAPTCHA Protection

### What I Added

We implemented rate limiting that locks accounts for 15 minutes after 5 failed login attempts and added Google reCAPTCHA v3 to block automated bot attacks.

### Why Rate Limiting Helps

Rate limiting and CAPTCHA protect against brute force attacks by slowing down automated password guessing and preventing bots from submitting thousands of login attempts per minute.

### Rate Limiting Implementation

Failed login attempts are tracked with timestamps and trigger a 15-minute lockout after 5 failures. reCAPTCHA analyzes user behavior like mouse movements and typing patterns to assign human scores that determine if submissions are processed.

### Rate Limiting Configuration

```javascript
const RATE_LIMIT_CONFIG = {
  MAX_FAILED_ATTEMPTS: 5,
  LOCKOUT_DURATION_MINUTES: 15,
  LOCKOUT_DURATION_MS: 15 * 60 * 1000,
  RECAPTCHA_THRESHOLD: 0.5,
};
```

### Rate Limiting Features

- Max 5 failed attempts per email
- 15-minute lockout after limit reached
- Countdown timer display
- Auto-clear on successful login
- Attempts stored in localStorage with timestamps

### reCAPTCHA v3 Integration

- Site Key: `6LdZRhgsAAAAAA7o0EpsdmfO38VvnHxjNG6Ab0g2`
- Silent verification (no user interaction)
- Analyzes behavioral patterns
- Prevents bot signups and logins

### Lockout Warning UI

- Red warning box with countdown timer
- Clear message: "Account locked for X minutes"
- Attempt counter: "X attempts remaining"
- Mobile responsive design

---

## Security Status Dashboard

| Solution            | Status         | Coverage                                     |
| ------------------- | -------------- | -------------------------------------------- |
| Password Encryption | ✅ Implemented | All user passwords                           |
| Input Validation    | ✅ Implemented | All forms (login, signup, contact, checkout) |
| Session Management  | ✅ Implemented | 30-minute auto-timeout                       |
| Rate Limiting       | ✅ Implemented | 5 attempts / 15-minute lockout               |
| reCAPTCHA v3        | ✅ Implemented | All authentication forms                     |

---

## Threat Prevention Matrix

| Threat                  | Solution           | Status | Protection                         |
| ----------------------- | ------------------ | ------ | ---------------------------------- |
| Stealing User Passwords | Bcrypt Hashing     | ✅     | Passwords unreadable if exposed    |
| XSS Attacks             | Input Sanitization | ✅     | Script tags stripped, HTML escaped |
| SQL Injection           | Input Validation   | ✅     | Dangerous patterns rejected        |
| Session Hijacking       | Session Timeout    | ✅     | Auto-logout after 30 min           |
| Brute Force Attacks     | Rate Limiting      | ✅     | 5 attempts / 15-min lockout        |
| Bot Attacks             | reCAPTCHA v3       | ✅     | Behavioral analysis blocks bots    |
| Account Enumeration     | Rate Limiting      | ✅     | Same response for all emails       |

---

## Custom Configuration

To customize, edit `RATE_LIMIT_CONFIG` in `cincoscript.js`:

```javascript
const RATE_LIMIT_CONFIG = {
  MAX_FAILED_ATTEMPTS: 5, // Change to 3 for stricter
  LOCKOUT_DURATION_MINUTES: 15, // Change to 30 for longer
  RECAPTCHA_THRESHOLD: 0.5, // Change to 0.7 for stricter bot detection
};
```

---

## Initial Setup Required

1. Get reCAPTCHA v3 keys from: [https://www.google.com/recaptcha/admin](https://www.google.com/recaptcha/admin)
2. Replace `6LdZRhgsAAAAAA7o0EpsdmfO38VvnHxjNG6Ab0g2` in:
   - `logSign.html` (reCAPTCHA script)
   - `cincoscript.js` (in executeRecaptcha function - 2 places)
3. See `RECAPTCHA.md` for detailed instructions

---

## Security Monitoring

- Check Google reCAPTCHA Admin Console for bot traffic
- Monitor localStorage for failed attempt patterns
- Review lockout frequency in browser console

---

## Attack Timeline Example

| Time  | Event           | Result                            |
| ----- | --------------- | --------------------------------- |
| 0:00  | Attempt 1 fails | "4 attempts remaining"            |
| 0:05  | Attempt 2 fails | "3 attempts remaining"            |
| 0:10  | Attempt 3 fails | "2 attempts remaining"            |
| 0:15  | Attempt 4 fails | "1 attempt remaining"             |
| 0:20  | Attempt 5 fails | **Account locked for 15 minutes** |
| 0:21  | Try again       | "Account locked. 14:59 remaining" |
| 15:20 | After 15 min    | ✅ Lockout cleared, can try again |

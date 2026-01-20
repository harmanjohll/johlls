# Security Documentation
## Malay Language Adventure - Security Considerations and Best Practices

This document outlines security considerations for the Malay Language Adventure platform, current vulnerabilities, and recommended mitigation strategies.

---

## Table of Contents

1. [Security Overview](#security-overview)
2. [Current Security Issues](#current-security-issues)
3. [Risk Assessment](#risk-assessment)
4. [Immediate Actions Required](#immediate-actions-required)
5. [Long-Term Solutions](#long-term-solutions)
6. [Best Practices](#best-practices)
7. [Incident Response](#incident-response)

---

## Security Overview

### Current Architecture

The Malay Language Adventure is a **client-side single-page application** with:
- All code visible in browser
- API calls made directly from client
- Credentials embedded in source code
- No server-side security layer

⚠️ **This architecture has inherent security limitations** that must be understood and addressed.

---

## Current Security Issues

### 🚨 CRITICAL: Exposed API Keys

#### Issue #1: Google Gemini API Keys Exposed

**Location in learnmalay.html:**
- **Line ~400:** Quiz generation API key
  ```javascript
  const GEMINI_API_KEY = 'AIzaSyAn7vIqbxswzrW35CZC12SvEX3omjwwl2Q';
  ```
- **Line ~556:** Daily words generation API key
  ```javascript
  const DAILY_WORDS_API_KEY = 'AIzaSyC9AKQcORYqndRZGhvTh-OqyMvE-Q7F2aI';
  ```

**Vulnerability:**
- Keys are visible in browser source code
- Anyone can copy and use these keys
- Keys can be extracted and used elsewhere
- No rate limiting on client-side

**Potential Impact:**
- **Unauthorized API usage** - Others can make calls with your keys
- **Quota exhaustion** - Your free/paid quota consumed by others
- **Unexpected costs** - If on paid plan, you'll be charged for unauthorized usage
- **Service disruption** - Quota limits reached, app stops working

#### Issue #2: Supabase Credentials Exposed

**Location in learnmalay.html:**
- **Lines 69-70:**
  ```javascript
  const SUPABASE_URL = 'https://qvowmphqmflfanjospqm.supabase.co';
  const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InF2b3dtcGhxbWZsZmFuam9zcHFtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTI0NTY3NTksImV4cCI6MjA2ODAzMjc1OX0.-AdJqEXD8N5C4iNQH3W3s6e1swVompOMQh0rTj3gXQk';
  ```

**Vulnerability:**
- Supabase anon key visible in source code
- Database URL publicly accessible
- Without proper Row Level Security (RLS), data could be compromised

**Potential Impact:**
- **Data manipulation** - Unauthorized users could modify database
- **Data theft** - User scores and profiles could be stolen
- **Data deletion** - Malicious actors could delete records
- **Privacy breach** - Student usernames and progress exposed

**Mitigation Note:** The Supabase anon key is *designed* to be used client-side, BUT only if proper Row Level Security (RLS) policies are configured. Without RLS, this is a critical vulnerability.

---

## Risk Assessment

### Severity Levels

| Issue | Severity | Likelihood | Impact |
|-------|----------|------------|--------|
| Exposed Gemini API Keys | 🔴 **CRITICAL** | High | High |
| Exposed Supabase Key (No RLS) | 🔴 **CRITICAL** | High | High |
| Exposed Supabase Key (With RLS) | 🟡 **MEDIUM** | Medium | Low |
| No Rate Limiting | 🟡 **MEDIUM** | Medium | Medium |
| No Authentication | 🟡 **MEDIUM** | Low | Medium |

### Current Risk Level: 🔴 **HIGH**

The application is currently at high risk due to exposed API credentials without proper safeguards.

---

## Immediate Actions Required

### Action 1: Rotate All API Keys 🚨 **URGENT**

**These keys are now public in this repository and must be rotated immediately.**

#### Rotate Google Gemini API Keys

1. **Go to Google AI Studio / Cloud Console**
   - https://aistudio.google.com/apikey
   - Or Google Cloud Console → APIs & Services → Credentials

2. **Delete compromised keys**
   - Delete key: `AIzaSyAn7vIqbxswzrW35CZC12SvEX3omjwwl2Q`
   - Delete key: `AIzaSyC9AKQcORYqndRZGhvTh-OqyMvE-Q7F2aI`

3. **Generate new keys**
   - Create 2 new API keys
   - Set restrictions (see below)

4. **Apply API Key Restrictions**
   ```
   Application restrictions:
   - HTTP referrers (websites)
   - Add your domain(s): example.com, www.example.com, localhost

   API restrictions:
   - Restrict key to: Generative Language API
   ```

5. **Update learnmalay.html with new keys**
   - Replace old keys with new restricted keys
   - Do NOT commit these to public repository

#### Rotate Supabase Keys (If Needed)

1. **Check Row Level Security**
   - Go to Supabase Dashboard → Authentication → Policies
   - Verify RLS policies exist for `learners` table

2. **If NO RLS policies exist:**
   - **Immediately enable RLS** on `learners` table
   - Create policies (see Long-Term Solutions section)
   - Consider rotating the anon key for extra security

3. **If RLS policies DO exist:**
   - Anon key exposure is less critical (designed for client-side)
   - Still review policies to ensure they're restrictive enough

### Action 2: Implement Row Level Security (Supabase)

**Execute these SQL commands in Supabase SQL Editor:**

```sql
-- Enable RLS on learners table
ALTER TABLE learners ENABLE ROW LEVEL SECURITY;

-- Policy: Anyone can create a new learner profile
CREATE POLICY "Anyone can create learner"
ON learners FOR INSERT
WITH CHECK (true);

-- Policy: Anyone can read learner data (for leaderboards, if needed)
-- OR restrict to only own data if usernames should be private
CREATE POLICY "Anyone can read learners"
ON learners FOR SELECT
USING (true);

-- Policy: Users can only update their own score
-- This requires authentication to properly restrict
-- For now, allow updates but consider adding username validation
CREATE POLICY "Update own learner record"
ON learners FOR UPDATE
USING (true)
WITH CHECK (true);

-- Note: Without proper authentication, RLS is limited
-- Consider implementing authentication for better security
```

### Action 3: Add Rate Limiting (Client-Side)

While not foolproof, add basic rate limiting to reduce abuse:

```javascript
// Add to learnmalay.html
const rateLimiter = {
    quizCalls: [],
    maxCallsPerHour: 20,

    canMakeCall() {
        const now = Date.now();
        const oneHourAgo = now - (60 * 60 * 1000);

        // Remove calls older than 1 hour
        this.quizCalls = this.quizCalls.filter(time => time > oneHourAgo);

        if (this.quizCalls.length >= this.maxCallsPerHour) {
            return false;
        }

        this.quizCalls.push(now);
        return true;
    }
};

// Use before API calls
if (!rateLimiter.canMakeCall()) {
    alert('Terlalu banyak percubaan. Sila cuba lagi kemudian.');
    return;
}
```

### Action 4: Monitor API Usage

1. **Google Gemini API**
   - Go to Google Cloud Console → APIs & Services → Dashboard
   - Monitor daily usage
   - Set up billing alerts if on paid plan
   - Watch for unusual spikes

2. **Supabase**
   - Go to Supabase Dashboard → Settings → Usage
   - Monitor database operations
   - Check for unusual activity patterns

---

## Long-Term Solutions

### Solution 1: Backend Proxy Layer (RECOMMENDED)

**Architecture:**
```
Client → Backend API → Google Gemini / Supabase
```

**Implementation Options:**

#### Option A: Serverless Functions (Easiest)

Use **Vercel Functions**, **Netlify Functions**, or **Cloudflare Workers**:

**Example: Netlify Function**

```javascript
// netlify/functions/generate-quiz.js
const fetch = require('node-fetch');

exports.handler = async (event) => {
    // Get from environment variables
    const GEMINI_API_KEY = process.env.GEMINI_API_KEY;

    if (event.httpMethod !== 'POST') {
        return { statusCode: 405, body: 'Method Not Allowed' };
    }

    const { concept, tier } = JSON.parse(event.body);

    // Validate inputs
    if (!concept || tier === undefined) {
        return { statusCode: 400, body: 'Missing parameters' };
    }

    // Make API call with server-side key
    const response = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${GEMINI_API_KEY}`,
        {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                contents: [{ parts: [{ text: /* your prompt */ }] }]
            })
        }
    );

    const data = await response.json();

    return {
        statusCode: 200,
        body: JSON.stringify(data)
    };
};
```

**Update client-side code:**
```javascript
// Instead of calling Gemini directly
const response = await fetch('/.netlify/functions/generate-quiz', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ concept: 'imbuhan', tier: 1 })
});
```

**Benefits:**
- API keys hidden on server
- Can add authentication
- Can implement real rate limiting
- Can log and monitor usage

#### Option B: Express.js Backend

Create a Node.js/Express server:

```javascript
// server.js
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

app.post('/api/generate-quiz', async (req, res) => {
    // Similar to serverless function
    // API key from process.env.GEMINI_API_KEY
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

#### Option C: Supabase Edge Functions

Use Supabase's built-in edge functions:

```typescript
// supabase/functions/generate-quiz/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'

serve(async (req) => {
  const { concept, tier } = await req.json()
  const GEMINI_API_KEY = Deno.env.get('GEMINI_API_KEY')

  // Make API call and return result
})
```

### Solution 2: User Authentication

Implement proper authentication to:
- Restrict database access to authenticated users
- Enable better RLS policies
- Track individual user actions
- Prevent abuse

**Options:**
- **Supabase Auth** (recommended, already using Supabase)
- **Firebase Auth**
- **Auth0**
- **Custom JWT-based auth**

**Implementation with Supabase Auth:**

```javascript
// Registration with email/password
const { data, error } = await supabase.auth.signUp({
    email: 'user@example.com',
    password: 'secure-password'
});

// Better RLS policies possible:
CREATE POLICY "Users can only update own data"
ON learners FOR UPDATE
USING (auth.uid() = id);
```

### Solution 3: API Gateway with Rate Limiting

Use an API gateway service:
- **Kong**
- **AWS API Gateway**
- **Azure API Management**
- **Google Cloud Endpoints**

Benefits:
- Professional rate limiting
- Authentication/authorization
- Request throttling
- Usage analytics

---

## Best Practices

### For Current Setup

1. **Minimize Key Exposure**
   - Don't commit keys to public repositories
   - Use environment variables even in client-side (via build process)
   - Rotate keys regularly (monthly or after any suspected compromise)

2. **Monitor Continuously**
   - Check API usage daily
   - Set up alerts for unusual activity
   - Review Supabase logs weekly

3. **Educate Users**
   - Don't share the direct HTML file publicly
   - Host on a domain with referrer restrictions
   - Consider password-protecting the site during development

4. **Prepare for Compromise**
   - Have a key rotation procedure ready
   - Document all API keys and their locations
   - Keep backup access methods

### For Production Deployment

1. **NEVER** deploy with exposed API keys
2. **ALWAYS** use backend proxy for sensitive API calls
3. **IMPLEMENT** authentication before public launch
4. **ENABLE** RLS on all Supabase tables
5. **USE** environment variables for all secrets
6. **SET UP** monitoring and alerting
7. **CONDUCT** security audits regularly
8. **HAVE** incident response plan ready

### Secure Development Workflow

```bash
# Use .env files (never commit these)
.env.local:
GEMINI_API_KEY=your-key-here
SUPABASE_KEY=your-key-here

# Add to .gitignore
.env
.env.local
.env.production
```

```javascript
// Use build-time environment variables
const GEMINI_API_KEY = import.meta.env.VITE_GEMINI_API_KEY;
```

---

## Incident Response

### If API Keys Are Compromised

1. **Immediate Actions** (within 1 hour)
   - [ ] Disable/delete compromised keys
   - [ ] Generate new keys with restrictions
   - [ ] Update application with new keys
   - [ ] Deploy updated application

2. **Assessment** (within 24 hours)
   - [ ] Check API usage for unauthorized calls
   - [ ] Review billing for unexpected charges
   - [ ] Check database for unauthorized modifications
   - [ ] Identify scope of compromise

3. **Remediation** (within 1 week)
   - [ ] Implement backend proxy layer
   - [ ] Add authentication if not present
   - [ ] Enhance monitoring
   - [ ] Document incident and lessons learned

4. **Prevention** (ongoing)
   - [ ] Review security practices monthly
   - [ ] Conduct security audits quarterly
   - [ ] Update dependencies regularly
   - [ ] Train team on security best practices

### If Database Is Compromised

1. **Immediate Actions**
   - [ ] Enable RLS immediately
   - [ ] Revoke public access if needed
   - [ ] Backup current database state
   - [ ] Assess data integrity

2. **Recovery**
   - [ ] Restore from backup if data corrupted
   - [ ] Notify affected users if personal data exposed
   - [ ] Implement authentication
   - [ ] Add audit logging

---

## Security Checklist

### Pre-Deployment Checklist

- [ ] All API keys rotated and restricted
- [ ] No secrets in client-side code
- [ ] Backend proxy layer implemented
- [ ] Supabase RLS enabled and tested
- [ ] Authentication implemented (if handling sensitive data)
- [ ] Rate limiting in place
- [ ] Monitoring and alerts configured
- [ ] Security testing completed
- [ ] Incident response plan documented
- [ ] Team trained on security procedures

### Monthly Security Review

- [ ] Check API usage patterns
- [ ] Review Supabase logs
- [ ] Verify RLS policies still effective
- [ ] Test rate limiting
- [ ] Review user reports of issues
- [ ] Update dependencies
- [ ] Rotate API keys (quarterly)

---

## Additional Resources

### Security Documentation

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Supabase Security Best Practices](https://supabase.com/docs/guides/platform/security)
- [Google API Key Best Practices](https://cloud.google.com/docs/authentication/api-keys)

### Tools

- **GitHub Secret Scanning** - Detect committed secrets
- **Git-secrets** - Prevent committing secrets
- **Dependabot** - Monitor dependency vulnerabilities
- **Snyk** - Security scanning for dependencies

### Reporting Security Issues

If you discover a security vulnerability:

1. **DO NOT** open a public GitHub issue
2. Email security contact (to be set up)
3. Include:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

4. Allow time for fix before public disclosure

---

## Conclusion

Security is an ongoing process, not a one-time fix. The current application has critical vulnerabilities that must be addressed before production deployment.

**Priority Actions:**
1. ✅ **Rotate exposed API keys immediately**
2. ✅ **Enable Supabase RLS**
3. ✅ **Implement backend proxy layer**
4. ✅ **Add authentication**
5. ✅ **Set up monitoring**

Remember: **Security through obscurity is not security.** Even if you don't publicize your application, exposed keys can be found by automated scanners.

---

*Stay secure, stay vigilant, stay protected.* 🔒

# Security Pre-Deployment Checklist

Detailed security patterns and verification steps. Referenced by the security-review skill.

## 1. Secrets Management

- [ ] No hardcoded API keys, tokens, or passwords
- [ ] All secrets in environment variables
- [ ] `.env.local` in .gitignore
- [ ] No secrets in git history
- [ ] Production secrets in hosting platform

## 2. Input Validation

- [ ] All user inputs validated with schemas
- [ ] File uploads restricted (size, type, extension)
- [ ] No direct use of user input in queries
- [ ] Whitelist validation (not blacklist)
- [ ] Error messages don't leak sensitive info

### File Upload Example

```typescript
function validateFileUpload(file: File) {
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) throw new Error('File too large (max 5MB)')

  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) throw new Error('Invalid file type')

  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const ext = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!ext || !allowedExtensions.includes(ext)) throw new Error('Invalid extension')
}
```

## 3. SQL Injection Prevention

- [ ] All database queries use parameterized queries
- [ ] No string concatenation in SQL
- [ ] ORM/query builder used correctly

## 4. Authentication & Authorization

- [ ] Tokens stored in httpOnly cookies (not localStorage)
- [ ] Authorization checks before sensitive operations
- [ ] Row Level Security enabled where appropriate
- [ ] Role-based access control implemented
- [ ] Session management secure

## 5. XSS Prevention

- [ ] User-provided HTML sanitized (DOMPurify)
- [ ] CSP headers configured
- [ ] No unvalidated dynamic content rendering

### CSP Example

```typescript
const securityHeaders = [{
  key: 'Content-Security-Policy',
  value: "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://api.example.com;"
}]
```

## 6. CSRF Protection

- [ ] CSRF tokens on state-changing operations
- [ ] SameSite=Strict on all cookies

## 7. Rate Limiting

- [ ] Rate limiting on all API endpoints
- [ ] Stricter limits on expensive operations (search, auth)
- [ ] IP-based and user-based rate limiting

## 8. Sensitive Data Exposure

- [ ] No passwords, tokens, or secrets in logs
- [ ] Error messages generic for users, detailed only in server logs
- [ ] No stack traces exposed to users

## 9. Dependency Security

- [ ] Dependencies up to date
- [ ] No known vulnerabilities (`npm audit` clean)
- [ ] Lock files committed
- [ ] Dependabot or similar enabled

## 10. HTTPS & Headers

- [ ] HTTPS enforced in production
- [ ] CSP, X-Frame-Options configured
- [ ] CORS properly configured

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Web Security Academy](https://portswigger.net/web-security)

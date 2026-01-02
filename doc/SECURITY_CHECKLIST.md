# CRITICAL SECURITY CHECKLIST

## 🔴 IMMEDIATE ACTIONS REQUIRED - DO NOT SKIP

**Your codebase contains exposed credentials and security vulnerabilities that MUST be addressed before any production use.**

---

## 🚨 Priority 1: Exposed Credentials (FIX IMMEDIATELY)

### Found Hardcoded Credentials:

#### 1. Database Password Exposed
**Location:** `Presentation/Orca.Web/appsettings.Development.json`
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=136.144.214.127,1433;Initial Catalog=SustainableTravelDb;User ID=sa;Password=Vgij@5081"
  }
}
```

**Risk:** 🔴 CRITICAL - SA password exposed, production server IP visible

**Action Required:**
1. ✅ Immediately change the SQL Server password: `Vgij@5081`
2. ✅ Remove this file from version control (if applicable)
3. ✅ Use environment variables instead
4. ✅ Check if server `136.144.214.127` has been compromised

#### 2. Local Database Connection
**Location:** `Presentation/Orca.Web/App_Data/dataSettings.json`
```json
{
  "DataConnectionString": "Server=LFATY\\SQLEXPRESS;Initial Catalog=SustainableTravelDb;..."
}
```

**Risk:** 🟡 MEDIUM - Exposes local server name, no credentials but bad practice

**Action Required:**
1. ✅ Move to environment-specific configuration
2. ✅ Use User Secrets for development

#### 3. Weak JWT Secret
**Location:** `Presentation/Orca.Api/appsettings.json`
```json
{
  "JwtSettings": {
    "Secret": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
    "TokenLifetime": "00:00:45"
  }
}
```

**Risk:** 🔴 CRITICAL - Placeholder secret, anyone can forge tokens

**Action Required:**
1. ✅ Generate a strong cryptographic key (min 256 bits)
2. ✅ Store in secure configuration (Key Vault, env vars)
3. ✅ Increase token lifetime to reasonable value (1 hour recommended)

---

## 🔒 Priority 2: Security Configuration

### 1. Enable HTTPS Everywhere

**File:** `Presentation/Orca.Web/Startup.cs`

Ensure HTTPS redirection is enabled:
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseHttpsRedirection(); // ✅ This should be present
    app.UseHsts(); // ✅ Enable in production
}
```

### 2. Configure CORS Properly

**File:** Check CORS configuration in API startup

```csharp
// ❌ BAD - Allows all origins
services.AddCors(options => {
    options.AddPolicy("AllowAll", builder => {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});

// ✅ GOOD - Restrict to specific origins
services.AddCors(options => {
    options.AddPolicy("Production", builder => {
        builder.WithOrigins("https://yourdomain.com")
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});
```

### 3. Disable Detailed Errors in Production

**File:** `Presentation/Orca.Web/appsettings.json`

```json
{
  "Orca": {
    "DisplayFullErrorStack": false  // ✅ MUST be false in production
  }
}
```

---

## ⚙️ Priority 3: Environment-Based Configuration

### Implement User Secrets for Development

**Step 1:** Initialize User Secrets
```bash
cd Presentation/Orca.Web
dotnet user-secrets init
```

**Step 2:** Store sensitive data
```bash
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=SustainableTravelDb;Integrated Security=True"
dotnet user-secrets set "JwtSettings:Secret" "YOUR-STRONG-SECRET-KEY-HERE"
```

**Step 3:** Remove from appsettings.json
```json
{
  "ConnectionStrings": {
    // REMOVE ACTUAL CONNECTION STRING
    "DefaultConnection": "*** STORED IN USER SECRETS ***"
  },
  "JwtSettings": {
    "Secret": "*** STORED IN USER SECRETS ***",
    "TokenLifetime": "01:00:00"
  }
}
```

### Use Environment Variables in Production

**Docker Example:**
```yaml
environment:
  - ConnectionStrings__DefaultConnection=${DB_CONNECTION_STRING}
  - JwtSettings__Secret=${JWT_SECRET}
  - Orca__RedisCachingConnectionString=${REDIS_CONNECTION}
```

**Azure App Service Example:**
Add to Application Settings in Azure Portal:
- `ConnectionStrings:DefaultConnection`
- `JwtSettings:Secret`
- `Orca:RedisCachingConnectionString`

---

## 🔐 Priority 4: Authentication & Authorization

### 1. Password Policy

**File:** Check `Orca.Core/Domain/Users/UserSettings.cs`

Ensure strong password requirements:
```csharp
public class UserSettings
{
    public int PasswordMinLength { get; set; } = 8; // ✅ Minimum 8
    public bool PasswordRequireDigit { get; set; } = true; // ✅ Require number
    public bool PasswordRequireLowercase { get; set; } = true; // ✅ Require lowercase
    public bool PasswordRequireUppercase { get; set; } = true; // ✅ Require uppercase
    public bool PasswordRequireNonAlphanumeric { get; set; } = true; // ✅ Require special char
}
```

### 2. Implement Account Lockout

Prevent brute force attacks:
```csharp
public class UserSettings
{
    public int FailedPasswordAttemptThreshold { get; set; } = 5;
    public int FailedPasswordLockoutMinutes { get; set; } = 30;
}
```

### 3. Enable Two-Factor Authentication (Future Enhancement)

Add to backlog:
- SMS-based 2FA
- Authenticator app support
- Email verification codes

---

## 🛡️ Priority 5: Data Protection

### 1. Enable Data Protection

**File:** `Presentation/Orca.Web/Startup.cs`

```csharp
services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@".\keys"))
    .SetApplicationName("SustainableTravel")
    .SetDefaultKeyLifetime(TimeSpan.FromDays(90));
```

For production with Redis:
```csharp
services.AddDataProtection()
    .PersistKeysToStackExchangeRedis(redis, "DataProtection-Keys")
    .SetApplicationName("SustainableTravel");
```

### 2. Encrypt Sensitive Database Fields

Consider encrypting:
- User email addresses
- Phone numbers
- Payment information (if applicable)
- Personal identification data

### 3. GDPR Compliance

✅ The codebase already has GDPR features:
- User consent management
- Data export
- Right to be forgotten

Ensure these are properly configured in admin panel.

---

## 🔍 Priority 6: Input Validation

### 1. Anti-Forgery Tokens

**Check all forms have:**
```cshtml
<form asp-controller="..." asp-action="...">
    <orca-antiforgery-token />  @* ✅ Required *@
    <!-- form fields -->
</form>
```

### 2. SQL Injection Prevention

✅ Using Entity Framework = Protected by default
❌ Direct SQL queries = MUST use parameters

**Check for:**
```csharp
// ❌ DANGEROUS - SQL Injection risk
var sql = $"SELECT * FROM Users WHERE Email = '{email}'";

// ✅ SAFE - Parameterized query
var sql = "SELECT * FROM Users WHERE Email = @email";
dbContext.Users.FromSqlRaw(sql, new SqlParameter("@email", email));
```

### 3. XSS Prevention

✅ Razor views automatically encode output
❌ Check for `@Html.Raw()` usage - validate data first

---

## 📊 Priority 7: Logging & Monitoring

### 1. Enable Security Logging

Log authentication events:
- Failed login attempts
- Account lockouts
- Password changes
- Role changes
- Sensitive data access

### 2. Implement Security Headers

Add to `Startup.cs`:
```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("X-Frame-Options", "DENY");
    context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
    context.Response.Headers.Add("Referrer-Policy", "no-referrer");
    context.Response.Headers.Add("Content-Security-Policy", 
        "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';");
    await next();
});
```

### 3. Monitor for Suspicious Activity

Implement alerts for:
- Multiple failed login attempts
- Admin actions
- Database connection failures
- API rate limit violations

---

## 🌐 Priority 8: API Security

### 1. Rate Limiting

Implement API rate limiting to prevent abuse:
```csharp
// Add to API Startup
services.AddRateLimiting(options =>
{
    options.GeneralRules = new List<RateLimitRule>
    {
        new RateLimitRule
        {
            Endpoint = "*",
            Period = TimeSpan.FromMinutes(1),
            Limit = 60
        }
    };
});
```

### 2. API Key Management

If using API keys:
- ✅ Store hashed, not plaintext
- ✅ Implement key rotation
- ✅ Set expiration dates
- ✅ Allow per-key rate limits

### 3. Validate All Inputs

Every API endpoint should:
```csharp
[HttpPost]
public IActionResult Create([FromBody] ModelDto model)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState); // ✅ Validate first
    
    // Process...
}
```

---

## 📋 Security Checklist

### Before Development
- [ ] Change all exposed passwords in config files
- [ ] Generate new JWT secret (256-bit minimum)
- [ ] Set up User Secrets for local development
- [ ] Remove hardcoded credentials from code
- [ ] Enable HTTPS redirection
- [ ] Configure CORS properly

### Before Testing
- [ ] Implement password policy
- [ ] Enable account lockout
- [ ] Add anti-forgery tokens to all forms
- [ ] Configure security headers
- [ ] Test authentication flows
- [ ] Verify authorization on all endpoints

### Before Production
- [ ] Security audit completed
- [ ] Penetration testing performed
- [ ] All secrets in Key Vault/secure storage
- [ ] Disable detailed error messages
- [ ] Enable production logging
- [ ] Configure SSL/TLS certificates
- [ ] Set up monitoring and alerts
- [ ] Review and lock down firewall rules
- [ ] Implement rate limiting
- [ ] Enable DDoS protection
- [ ] Configure backup encryption
- [ ] Review database security
- [ ] Implement security incident response plan

---

## 🔧 Tools for Security Testing

### Recommended Security Scanners

1. **OWASP Dependency Check**
   ```bash
   # Check for vulnerable NuGet packages
   dotnet list package --vulnerable
   ```

2. **Security Code Scan**
   ```bash
   # Install security analyzer
   dotnet add package SecurityCodeScan.VS2019
   ```

3. **SonarQube** - For static analysis
4. **Burp Suite** - For penetration testing
5. **SQL Injection Tester** - Test database security

---

## 📞 Security Incident Response

### If You Suspect a Breach

1. **Immediate Actions:**
   - Change all passwords and API keys
   - Revoke all active sessions
   - Enable additional logging
   - Isolate affected systems

2. **Investigation:**
   - Review access logs
   - Check for unauthorized data access
   - Identify entry point
   - Document findings

3. **Remediation:**
   - Patch vulnerabilities
   - Reset compromised credentials
   - Notify affected users (if required by law)
   - Implement additional security measures

4. **Prevention:**
   - Conduct security review
   - Update security policies
   - Implement additional monitoring
   - Train team on security best practices

---

## 🎯 Quick Wins (Do These First)

### Top 5 Security Improvements (1 Day)

1. ✅ Change all passwords in config files
2. ✅ Generate and configure strong JWT secret
3. ✅ Set up User Secrets for development
4. ✅ Disable detailed errors in production config
5. ✅ Add security headers to responses

### Medium Priority (1 Week)

6. ✅ Implement environment-based configuration
7. ✅ Set up Azure Key Vault (or equivalent)
8. ✅ Configure password policy
9. ✅ Enable account lockout
10. ✅ Implement API rate limiting

### Ongoing Security

- Regular security audits (quarterly)
- Keep dependencies updated
- Monitor security advisories
- Train development team
- Penetration testing (annually)

---

## 📚 Resources

### Security Guidelines
- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **ASP.NET Core Security:** https://docs.microsoft.com/aspnet/core/security/
- **Azure Security Best Practices:** https://docs.microsoft.com/azure/security/

### Tools
- **OWASP ZAP:** https://www.zaproxy.org/
- **Burp Suite Community:** https://portswigger.net/burp/communitydownload
- **dotnet-retire:** https://github.com/RetireNet/dotnet-retire

---

## ⚠️ FINAL WARNING

**DO NOT deploy this application to production without addressing ALL Priority 1 and Priority 2 items.**

The exposed credentials and weak secrets pose an immediate security risk. Any deployment before fixing these issues could result in:
- Data breach
- Unauthorized access
- Financial loss
- Legal liability
- Reputation damage

**Estimated Time to Secure:**
- Priority 1: 2-4 hours
- Priority 2: 1-2 days
- Priority 3-4: 1 week
- Full security hardening: 2-3 weeks

---

*Last Updated: October 14, 2025*


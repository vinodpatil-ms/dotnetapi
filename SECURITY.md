# Security Configuration Guide

## Database Connection Security

This application implements secure database credential management following OWASP security guidelines.

### Development Environment

**User Secrets** are used for development to keep sensitive data out of source control:

```bash
# Initialize User Secrets (already done)
dotnet user-secrets init --project webapi

# Set your database connection string
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=your-server;Database=custorders;User Id=your-user;Password=your-password;TrustServerCertificate=True;" --project webapi

# List all secrets (for verification)
dotnet user-secrets list --project webapi
```

### Production Environment

**Environment Variables** should be used for production deployments:

```bash
# Set environment variable for production
export DATABASE_CONNECTION_STRING="Server=prod-server;Database=custorders;User Id=prod-user;Password=secure-password;TrustServerCertificate=True;"

# Or in Docker/Container environments
docker run -e DATABASE_CONNECTION_STRING="Server=..." your-app
```

### Configuration Priority

The application loads database credentials in this order:
1. **Environment Variable**: `DATABASE_CONNECTION_STRING`
2. **User Secrets**: `ConnectionStrings:DefaultConnection`
3. **Configuration File**: `appsettings.json` (should not contain sensitive data)

### Azure Key Vault (Recommended for Production)

For enterprise production deployments, use Azure Key Vault:

```csharp
// Add to Program.cs for Azure Key Vault integration
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{keyVaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
```

## Security Best Practices

### ✅ DO
- Use User Secrets for development
- Use Environment Variables or Azure Key Vault for production
- Rotate database passwords regularly
- Use Windows Authentication/Integrated Security when possible
- Review connection strings before committing code
- Use principle of least privilege for database users

### ❌ DON'T
- Store passwords in appsettings.json
- Commit secrets to source control
- Share User Secrets between developers
- Use default/weak passwords
- Grant excessive database permissions

## Verification

Verify no hardcoded credentials exist:

```bash
# Search for potential credential patterns
grep -r "Password=" . --exclude-dir=bin --exclude-dir=obj --exclude=SECURITY.md
grep -r "pwd=" . --exclude-dir=bin --exclude-dir=obj --exclude=SECURITY.md
```

## Security Incident Response

If credentials are accidentally committed:
1. **Immediately rotate the password**
2. Review git history for exposure
3. Consider using tools like BFG Repo-Cleaner for history cleanup
4. Update all environments with new credentials
5. Review access logs for unauthorized usage

## Compliance

This configuration addresses:
- **OWASP A02:2021** - Cryptographic Failures
- **OWASP A05:2021** - Security Misconfiguration
- **SOX, PCI DSS** compliance requirements
- Industry security best practices
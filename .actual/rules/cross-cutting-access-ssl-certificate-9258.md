# Use .get() Method with Default Values for Safe External Header and Environment Access: Access Ssl Certificate

These rules are ALWAYS ACTIVE for all files accessing HTTP headers, environment variables, and SSL certificate dictionaries from external sources.

### Rules

- **R-SSL-001** MUST: All access to SSL certificate dictionaries (subjectAltName, subject) MUST use .get() with empty tuple or appropriate default

### Verify

```bash
# Check for direct dictionary access to headers without .get()
grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal

# Check for direct dictionary access to os.environ without .get()
grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal

# Verify .get() usage for external data access patterns
grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l  # Should match pattern usage
```

**Accept when:**
- All SSL certificate dictionary access in src/urllib3/ uses .get() method with appropriate defaults (empty tuple for subjectAltName and subject)
- All HTTP header access uses .get() method with appropriate defaults
- All os.environ access uses .get() method with appropriate defaults
- No KeyError exceptions occur in production logs related to certificate, header, or environment variable access
- Default values maintain type consistency with present values

<enforcement>
Clause Code MUST NOT skip or defer verification. All SSL certificate dictionary access patterns must be validated before merge.
</enforcement>
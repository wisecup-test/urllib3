# Use .get() Method with Default Values for Safe External Header and Environment Access: Access Http Response

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-HTTP-001** MUST: All access to HTTP response headers from external clients MUST use the .get() method with an appropriate default value rather than direct dictionary access.

### Verify

```bash
# Check for direct dictionary access to headers without .get()
grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal

# Check for direct dictionary access to os.environ without .get()
grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal

# Verify .get() usage for external data access
grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l  # Should match pattern usage
```

**Accept when:**
- All HTTP header access in src/urllib3/ uses .get() method with appropriate defaults
- All os.environ access uses .get() method with appropriate defaults
- No KeyError exceptions occur in production logs related to header or environment variable access
- For HTTP headers, None is used as default when absence is semantically meaningful (e.g., response.headers.get('Retry-After'))
- For environment variables, empty string or False is used as default based on downstream usage (e.g., os.environ.get('CI'))
- For certificate fields, empty tuple () is used as default to enable safe iteration (e.g., cert.get('subjectAltName', ()))
- For lookup tables and mappings, fallback values maintain type consistency (e.g., port_by_scheme.get(scheme, 80))

<enforcement>
Clause EXC-001 permits direct access only when code explicitly validates key presence before access and handles KeyError in an exception handler. Claude Code MUST NOT skip or defer verification of this rule.
</enforcement>
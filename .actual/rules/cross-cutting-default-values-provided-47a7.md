# Use .get() Method with Default Values for Safe External Header and Environment Access: Default Values Provided

These rules are ALWAYS ACTIVE for all code that accesses external HTTP headers, environment variables, SSL certificate dictionaries, and configuration lookup tables where keys may be absent.

### Rules

- **R-EXT-001** SHOULD: Default values provided to .get() SHOULD be type-appropriate (empty string, False, None, 0, empty tuple) based on downstream usage.
- **R-EXT-002** MUST: Use .get() method for all HTTP response headers received from external servers to prevent KeyError exceptions.
- **R-EXT-003** MUST: Use .get() method for all HTTP request headers received from external clients to prevent KeyError exceptions.
- **R-EXT-004** MUST: Use .get() method for all environment variables read from os.environ to prevent KeyError exceptions.
- **R-EXT-005** MUST: Use .get() method for SSL certificate dictionaries from external TLS handshakes to prevent KeyError exceptions.
- **R-EXT-006** SHOULD: For HTTP headers, use None as default when absence is semantically meaningful (e.g., response.headers.get('Retry-After') returns None to indicate no retry guidance).
- **R-EXT-007** SHOULD: For environment variables, use empty string or False as default based on downstream boolean vs. string usage (e.g., os.environ.get('CI') for boolean checks).
- **R-EXT-008** SHOULD: For certificate fields, use empty tuple () as default to enable safe iteration (e.g., cert.get('subjectAltName', ())).
- **R-EXT-009** SHOULD: For lookup tables and mappings, provide fallback values that maintain type consistency (e.g., port_by_scheme.get(scheme, 80) for integer ports).
- **R-EXT-010** MAY: Exception EXC-001 applies when code explicitly validates key presence before access and handles KeyError in an exception handler; document the specific case in code comments and obtain maintainer approval.

### Verify

```bash
# Check for direct dictionary access to headers without .get()
grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l
# Expected: 0 or minimal

# Check for direct dictionary access to os.environ without .get()
grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l
# Expected: 0 or minimal

# Verify .get() usage for external data access
grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l
# Expected: matches pattern usage across 14+ files
```

**Accept when:**
- All HTTP header access in src/urllib3/ uses .get() method with appropriate defaults
- All os.environ access uses .get() method with appropriate defaults
- No KeyError exceptions occur in production logs related to header or environment variable access
- Default values are type-appropriate and match downstream usage expectations
- Certificate field access uses empty tuple () as default for safe iteration

<enforcement>
Clause Code MUST NOT skip or defer verification. Static analysis with grep patterns MUST run in CI pipeline checking for direct dictionary access to headers and environ. Code review MUST block merge if external data structures use direct access without documented justification and exception approval.
</enforcement>
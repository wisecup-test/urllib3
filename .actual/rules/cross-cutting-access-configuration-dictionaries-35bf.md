# Use .get() Method with Default Values for Safe External Header and Environment Access: Access Configuration Dictionaries

These rules are ALWAYS ACTIVE for all code accessing external HTTP headers, environment variables, SSL certificate dictionaries, and configuration lookup tables across urllib3 core modules, connection pools, response handlers, and test infrastructure.

### Rules

- **R-CONFIG-001** SHOULD: Access to configuration dictionaries and lookup tables (port_by_scheme, HASHFUNC_MAP, _SSL_VERSION_TO_TLS_VERSION) SHOULD use .get() with fallback values.
- **R-CONFIG-002** MUST: HTTP response headers received from external servers MUST use .get() method with appropriate defaults (e.g., response.headers.get('Retry-After')).
- **R-CONFIG-003** MUST: HTTP request headers received from external clients MUST use .get() method with appropriate defaults.
- **R-CONFIG-004** MUST: Environment variables read from os.environ MUST use .get() method with appropriate defaults (e.g., os.environ.get('CI')).
- **R-CONFIG-005** MUST: SSL certificate dictionaries from external TLS handshakes MUST use .get() method with appropriate defaults (e.g., cert.get('subjectAltName', ())).
- **R-CONFIG-006** SHOULD: For HTTP headers, use None as default when absence is semantically meaningful.
- **R-CONFIG-007** SHOULD: For environment variables, use empty string or False as default based on downstream boolean vs. string usage.
- **R-CONFIG-008** SHOULD: For certificate fields, use empty tuple () as default to enable safe iteration.
- **R-CONFIG-009** SHOULD: For lookup tables and mappings, provide fallback values that maintain type consistency.
- **R-CONFIG-010** MAY: Direct dictionary access with try/except KeyError blocks is permitted only when KeyError itself carries semantic meaning that must be logged or handled distinctly, with explicit documentation in code comments.

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
# Expected: Should match pattern usage across 14+ files
```

**Accept when:**
- All HTTP header access in src/urllib3/ uses .get() method with appropriate defaults
- All os.environ access uses .get() method with appropriate defaults
- No KeyError exceptions occur in production logs related to header or environment variable access
- Default values are type-consistent with actual values when keys are present
- Optional vs. required fields are clearly documented

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST check for direct dictionary access patterns. CI pipeline MUST fail if direct dictionary access to headers or environ is detected in new code without documented justification and explicit KeyError handling.
</enforcement>
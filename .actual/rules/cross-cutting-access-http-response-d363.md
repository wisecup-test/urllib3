# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Access Http Response

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-DICT-GET-001** MUST: All access to HTTP response headers MUST use the .get() method with appropriate default values rather than direct key access.

### Verify

```bash
# Count .get() usage on headers, environ, cert, kwargs across production and test code
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct dictionary access patterns that should use .get()
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite to verify no KeyError exceptions from header/config access
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -i 'keyerror.*header\|keyerror.*environ' || echo 'No KeyError from header/config access detected'
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Canonical default values are established for common protocol-level keys (e.g., port_by_scheme.get(scheme, 80) for HTTP)
- Boolean configuration flags use False as default (e.g., self.headers.get(sort_key, False))
- Optional string headers use .get() without default argument to return None, with explicit None-checking before processing
- Test code uses .get() with explicit defaults matching production behavior

<enforcement>
Clause Code MUST NOT skip or defer verification. All HTTP response header access patterns must be reviewed for .get() compliance before merge approval. Static analysis via grep or AST-based linting must detect direct dictionary access on known external data structures. Test suite execution must be monitored for KeyError exceptions originating from header or configuration access.
</enforcement>
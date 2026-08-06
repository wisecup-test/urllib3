# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Internal Data Structure

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-DICT-001** SHOULD: Internal data structure access (self.headers.get, kwargs.get) SHOULD prefer .get() method when the key may legitimately be absent.

### Verify

```bash
# Count .get() usage on headers, environ, cert, kwargs
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct bracket access on external data structures
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite and verify no KeyError from dictionary access at boundaries
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -c 'KeyError.*headers\|KeyError.*environ\|KeyError.*cert' || echo "0"
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Certificate field access during SSL hostname matching uses .get() for optional fields
- Configuration parameter access in retry logic and proxy handling uses .get() for optional values

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes that modify dictionary access patterns on external data structures (headers, environ, cert, kwargs). Violations require code review feedback requesting change to .get() method before merge approval.
</enforcement>
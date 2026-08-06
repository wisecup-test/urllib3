# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Certificate Dictionary Access

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-CERT-001** SHOULD: Certificate dictionary access (cert.get) SHOULD use .get() method when accessing optional fields like subjectAltName and subject.

### Verify

```bash
# Count .get() usage on headers, environ, cert, and kwargs across production and test code
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct bracket access on external data structures (should be minimal)
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite to verify no KeyError exceptions from dictionary access at boundaries
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -c 'KeyError.*headers\|KeyError.*environ\|KeyError.*cert' || echo "0"
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Certificate field access during SSL hostname matching uses .get() for optional fields

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST execute successfully before accepting changes that modify certificate dictionary access patterns or external data boundary handling.
</enforcement>
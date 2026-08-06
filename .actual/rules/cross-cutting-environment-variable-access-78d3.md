# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Environment Variable Access

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-ENV-001** MUST: Environment variable access via os.environ MUST use .get() method to handle missing configuration gracefully.

### Verify

```bash
# Count .get() usage on headers, environ, cert, kwargs across production and test code
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct bracket access on external data structures (should be minimal)
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite to verify no KeyError exceptions from dictionary access at boundaries
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -c 'KeyError.*headers\|KeyError.*environ' || echo "0"
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Direct bracket access on external data structures (headers, environ, cert) is minimal or absent

<enforcement>
Clause Code MUST NOT skip or defer verification. All environment variable access patterns must be reviewed for .get() compliance before merge approval. Static analysis via grep or AST-based linting must detect direct dictionary access on known external data structures. Violations require code review feedback requesting change to .get() method. Exceptions require inline comment documentation, code review approval, and test coverage demonstration.
</enforcement>
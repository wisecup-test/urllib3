# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Test Code Use

These rules are ALWAYS ACTIVE for all HTTP response header access in production code and integration test code at external client boundaries.

### Rules

- **R-SAFE-001** MUST: Test code MUST use .get() method when validating headers in integration tests with dummy servers or external endpoints
- **R-SAFE-002** MUST: Production code MUST use .get() method when accessing HTTP response headers (e.g., response.headers.get('Retry-After'))
- **R-SAFE-003** MUST: Environment variable access at external boundaries MUST use os.environ.get() with appropriate defaults
- **R-SAFE-004** MUST: Port and scheme lookups from external configuration MUST use .get() with explicit defaults (e.g., port_by_scheme.get(scheme, 80))
- **R-SAFE-005** SHOULD: Provide explicit defaults when appropriate rather than relying on None returns from .get()
- **R-SAFE-006** SHOULD: Include explicit None checks in critical paths and add type hints to catch None handling issues
- **R-SAFE-007** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling

### Verify

```bash
# Check for direct dictionary access to headers (should return 0)
grep -r '\.headers\[' src/urllib3/ test/ --include='*.py' | grep -v '.get(' | wc -l

# Check for widespread .get() usage at external boundaries (should return >50)
grep -r 'response\.headers\.get\|returned_headers\.get\|os\.environ\.get' src/urllib3/ test/ --include='*.py' | wc -l

# Run integration tests to validate header access patterns
python -m pytest test/test_connectionpool.py test/with_dummyserver/test_poolmanager.py -v
```

**Accept when:**
- First verify command returns 0 (no direct dictionary access to headers in production or test code)
- Second verify command returns >50 (widespread .get() usage at external boundaries)
- All integration tests pass without KeyError exceptions from header access

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting code that modifies header access patterns.
</enforcement>
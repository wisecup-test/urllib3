# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Code Accessing Headers

These rules are ALWAYS ACTIVE for all code accessing headers at external client boundaries, including HTTP response header access in production code, integration test header validation, environment variable access, and port/scheme lookups from external configuration.

### Rules

- **R-HDR-001** MUST: Code accessing headers at external client boundaries (HTTP responses, environment variables via os.environ) MUST use safe access patterns with .get() rather than direct dictionary access.
- **R-HDR-002** MUST: HTTP response header access in production code (connectionpool.py, retry.py, response.py) MUST use response.headers.get('Header-Name') pattern.
- **R-HDR-003** MUST: Integration test header validation MUST use .get() when accessing headers returned from dummy servers.
- **R-HDR-004** MUST: Environment variable access at external boundaries MUST use os.environ.get() rather than direct access.
- **R-HDR-005** MUST: Port and scheme lookups from external configuration MUST use port_by_scheme.get(scheme, default) pattern.
- **R-HDR-006** SHOULD: Provide explicit defaults when appropriate (e.g., port_by_scheme.get(scheme, 80)) rather than relying on None.
- **R-HDR-007** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling.
- **R-HDR-008** SHOULD: Include explicit None checks in critical paths and add type hints to catch None handling issues.

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
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting code that modifies header access patterns.
</enforcement>
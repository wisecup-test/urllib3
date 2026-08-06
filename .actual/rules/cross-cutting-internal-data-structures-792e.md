# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Internal Data Structures

These rules are ALWAYS ACTIVE for HTTP response header access in production code, integration test header validation, environment variable access at external boundaries, and port/scheme lookups from external configuration.

### Rules

- **R-SAFE-001** MUST: Use `.get()` with appropriate defaults for all HTTP response header access in production code (connectionpool.py, retry.py, response.py).
- **R-SAFE-002** MUST: Use `.get()` for environment variable access at external boundaries (e.g., `os.environ.get('CI')`).
- **R-SAFE-003** MUST: Use `.get()` for port and scheme lookups from external configuration (e.g., `port_by_scheme.get(scheme, 80)`).
- **R-SAFE-004** SHOULD: Provide explicit defaults when appropriate rather than relying on None returns.
- **R-SAFE-005** SHOULD: Include explicit None checks in critical paths where missing headers could cause downstream errors.
- **R-SAFE-006** MAY: Use direct dictionary access for internal data structures where key presence is guaranteed by construction.
- **R-SAFE-007** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling.

### Verify

```bash
# Check for direct dictionary access to headers in production and test code
grep -r '\.headers\[' src/urllib3/ test/ --include='*.py' | grep -v '.get(' | wc -l

# Verify widespread .get() usage at external boundaries
grep -r 'response\.headers\.get\|returned_headers\.get\|os\.environ\.get' src/urllib3/ test/ --include='*.py' | wc -l

# Run integration tests to validate header access patterns
python -m pytest test/test_connectionpool.py test/with_dummyserver/test_poolmanager.py -v
```

**Accept when:**
- First verify command returns 0 (no direct dictionary access to headers in production or test code)
- Second verify command returns >50 (widespread .get() usage at external boundaries)
- All integration tests pass without KeyError exceptions from header access

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes that modify header access patterns.
</enforcement>
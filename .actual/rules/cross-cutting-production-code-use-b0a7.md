# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Production Code Use

These rules are ALWAYS ACTIVE for all production code and integration tests that access HTTP response headers from external clients, environment variables, or external configuration sources.

### Rules

- **R-SAFE-001** MUST: Production code MUST use .get() method with appropriate defaults when accessing response headers from external clients.
- **R-SAFE-002** MUST: Use response.headers.get('Header-Name') for all HTTP response header access in production code.
- **R-SAFE-003** SHOULD: Provide explicit defaults when appropriate (e.g., port_by_scheme.get(scheme, 80)) rather than relying on None.
- **R-SAFE-004** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling.
- **R-SAFE-005** MUST: Environment variable access at external boundaries MUST use os.environ.get() with appropriate defaults.
- **R-SAFE-006** MUST: Port and scheme lookups from external configuration MUST use .get() with appropriate defaults.
- **R-SAFE-007** SHOULD: In test code, use assertions that handle None: assert returned_headers.get('Foo') == expected_value or returned_headers.get('Foo') is None.

### Verify

```bash
# Check for direct dictionary access to headers in production or test code
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
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting code that modifies response header access patterns.
</enforcement>
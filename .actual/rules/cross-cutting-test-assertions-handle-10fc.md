# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Test Assertions Handle

These rules are ALWAYS ACTIVE for HTTP response header access in production code, integration test header validation, environment variable access at external boundaries, and port/scheme lookups from external configuration.

### Rules

- **R-SAFE-001** SHOULD: Test assertions SHOULD handle None returns from .get() calls explicitly to distinguish between missing headers and incorrect values.
- **R-SAFE-002** MUST: Use response.headers.get('Header-Name') for all HTTP response header access in production code rather than direct dictionary access.
- **R-SAFE-003** SHOULD: Provide explicit defaults when appropriate (e.g., port_by_scheme.get(scheme, 80)) rather than relying on None.
- **R-SAFE-004** SHOULD: In test code, use assertions that handle None: assert returned_headers.get('Foo') == expected_value or returned_headers.get('Foo') is None.
- **R-SAFE-005** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling.

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
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to header access patterns.
</enforcement>
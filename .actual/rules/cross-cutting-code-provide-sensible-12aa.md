# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Code Provide Sensible

These rules are ALWAYS ACTIVE for HTTP response header access in production code, integration test header validation, environment variable access at external boundaries, and port/scheme lookups from external configuration.

### Rules

- **R-SAFE-001** SHOULD: Code SHOULD provide sensible defaults to .get() calls (e.g., port_by_scheme.get(scheme, 80)) rather than relying on None returns.
- **R-SAFE-002** MUST: Use response.headers.get('Header-Name') for all HTTP response header access in production code at external client boundaries.
- **R-SAFE-003** SHOULD: Provide explicit defaults when appropriate (e.g., port_by_scheme.get(scheme, 80)) rather than relying on None.
- **R-SAFE-004** MUST: Avoid direct dictionary access (headers['Location']) at external client boundaries; use .get() instead.
- **R-SAFE-005** SHOULD: In test code, use assertions that handle None: assert returned_headers.get('Foo') == expected_value or returned_headers.get('Foo') is None.
- **R-SAFE-006** SHOULD: Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling.

### Verify

```bash
# Verify no direct dictionary access to headers in production or test code
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
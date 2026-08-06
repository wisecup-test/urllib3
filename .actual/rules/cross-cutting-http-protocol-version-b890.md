# Establish HTTP Client Compatibility Testing for Public API Contracts: Http Protocol Version

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and protocol version compatibility validation in the codebase.

### Rules

- **R-HTTP-001** MUST: HTTP protocol version compatibility (HTTP/2) MUST be verified through dedicated test functions (test_h2_version_check).
- **R-HTTP-002** MUST: All public-facing HTTP API endpoints and contracts MUST include corresponding compatibility tests in test/test_compatibility.py.
- **R-HTTP-003** MUST: Compatibility tests MUST validate at least urllib, urllib3, and http.cookiejar client implementations.
- **R-HTTP-004** SHOULD: Use pytest fixtures to parameterize tests across multiple HTTP client implementations, reducing code duplication.
- **R-HTTP-005** SHOULD: Use unittest.mock.patch for service boundary isolation to ensure tests remain fast and deterministic.
- **R-HTTP-006** SHOULD: Document supported HTTP client libraries and protocol versions in API documentation, referencing compatibility test coverage.

### Verify

```bash
# Verify compatibility test file exists and contains required test functions
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Verify test collection includes HTTP/2 and extraction tests
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in compatibility tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py

# Run all compatibility tests
python -m pytest test/test_compatibility.py -v
```

**Accept when:**
- test/test_compatibility.py exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions (test_h2_version_check)
- All compatibility tests pass in CI pipeline before deployment
- New public API endpoints include corresponding compatibility tests before merge approval

<enforcement>
Claude Code MUST NOT skip or defer verification of HTTP client compatibility testing. All public API endpoints MUST have corresponding compatibility tests. CI pipeline MUST fail if compatibility tests are missing or failing. Pull requests adding or modifying public APIs MUST include compatibility test updates before merge approval.
</enforcement>
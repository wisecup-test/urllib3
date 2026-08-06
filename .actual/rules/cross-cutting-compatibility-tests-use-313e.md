# Establish HTTP Client Compatibility Testing for Public API Contracts: Compatibility Tests Use

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and HTTP protocol version compatibility validation in the codebase.

### Rules

- **R-COMPAT-001** SHOULD: Compatibility tests SHOULD use pytest as the test framework for consistency with existing test infrastructure.
- **R-COMPAT-002** MUST: All compatibility tests MUST be placed in `test/test_compatibility.py` following the established pattern with `test_extract` and `test_h2_version_check` as examples.
- **R-COMPAT-003** MUST: Compatibility tests MUST validate at least urllib, urllib3, and http.cookiejar client implementations.
- **R-COMPAT-004** MUST: HTTP/2 version compatibility MUST be explicitly tested through dedicated test functions.
- **R-COMPAT-005** SHOULD: Pytest fixtures SHOULD be used to parameterize tests across multiple HTTP client implementations, reducing code duplication.
- **R-COMPAT-006** SHOULD: unittest.mock.patch SHOULD be used for service boundary isolation to ensure tests remain fast and deterministic.
- **R-COMPAT-007** MUST: New public API endpoints MUST include corresponding compatibility tests before merge approval.
- **R-COMPAT-008** MUST: Supported HTTP client libraries and protocol versions MUST be documented in API documentation, referencing compatibility test coverage.

### Verify

```bash
# Verify compatibility test file exists and contains required test functions
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Collect and display all compatibility tests
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in compatibility tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py

# Run all compatibility tests
python -m pytest test/test_compatibility.py -v
```

**Accept when:**
- `test/test_compatibility.py` exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions
- All compatibility tests pass in CI pipeline before deployment
- New public API endpoints include corresponding compatibility tests
- Supported HTTP client libraries and protocol versions are documented in API documentation

<enforcement>
Claude Code MUST NOT skip or defer verification of compatibility test presence and execution. Pull requests adding or modifying public APIs MUST include compatibility test updates. CI pipeline MUST fail if compatibility tests are missing for new public API endpoints.
</enforcement>
# Establish HTTP Client Compatibility Testing for Public API Contracts: Additional Http Client

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and protocol version compatibility validation in the codebase.

### Rules

- **R-COMPAT-001** MAY: Additional HTTP client libraries MAY be added to the compatibility test suite as new dependencies are introduced.
- **R-COMPAT-002** MUST: All compatibility tests MUST be placed in `test/test_compatibility.py` following the established pattern with `test_extract` and `test_h2_version_check` as examples.
- **R-COMPAT-003** MUST: Tests MUST validate at least urllib, urllib3, and http.cookiejar client implementations.
- **R-COMPAT-004** MUST: HTTP/2 version compatibility MUST be explicitly tested through dedicated test functions.
- **R-COMPAT-005** SHOULD: Use pytest fixtures to parameterize tests across multiple HTTP client implementations, reducing code duplication.
- **R-COMPAT-006** SHOULD: Implement test classes (e.g., TestCookiejar, TestInitialization) to organize related compatibility test cases by functional area.
- **R-COMPAT-007** MUST: Use `unittest.mock.patch` for service boundary isolation to ensure tests remain fast and deterministic without external dependencies.
- **R-COMPAT-008** MUST: Document supported HTTP client libraries and protocol versions in API documentation, referencing compatibility test coverage.
- **R-COMPAT-009** MUST: New public API endpoints MUST include corresponding compatibility tests before merge approval.
- **R-COMPAT-010** MUST: Compatibility tests MUST pass in CI pipeline before deployment.

### Verify

```bash
# Verify compatibility test file exists and contains required test functions
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Collect and display all compatibility tests
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py

# Run all compatibility tests
python -m pytest test/test_compatibility.py -v
```

**Accept when:**
- `test/test_compatibility.py` exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions (e.g., `test_h2_version_check`)
- All compatibility tests pass in CI pipeline before deployment
- New public API endpoints include corresponding compatibility tests
- Service boundary mocking uses `unittest.mock.patch` for isolation
- Supported HTTP client libraries and protocol versions are documented in API documentation

<enforcement>
Claude Code MUST NOT skip or defer verification of compatibility test coverage. All public API endpoints MUST have corresponding compatibility tests. CI pipeline MUST fail if compatibility tests are missing or failing. Pull requests adding or modifying public APIs MUST include compatibility test updates before merge approval.
</enforcement>
# Establish HTTP Client Compatibility Testing for Public API Contracts: Service Boundary Definitions

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and service boundary definitions in the codebase.

### Rules

- **R-SBD-001** MUST: Service boundary definitions MUST be tested using mock and patch techniques to isolate external dependencies.
- **R-SBD-002** MUST: All public-facing HTTP API endpoints and contracts MUST include compatibility tests validating at least urllib, urllib3, and http.cookiejar client implementations.
- **R-SBD-003** MUST: HTTP protocol version compatibility (HTTP/1.1, HTTP/2) MUST be explicitly tested through dedicated test functions.
- **R-SBD-004** MUST: Compatibility tests MUST be placed in test/test_compatibility.py following the established pattern with test_extract and test_h2_version_check as examples.
- **R-SBD-005** SHOULD: Use pytest fixtures to parameterize tests across multiple HTTP client implementations to reduce code duplication.
- **R-SBD-006** SHOULD: Organize related compatibility test cases by functional area using dedicated test classes (e.g., TestCookiejar, TestInitialization).
- **R-SBD-007** SHOULD: Document supported HTTP client libraries and protocol versions in API documentation, referencing compatibility test coverage.

### Verify

```bash
# Verify compatibility test file exists and contains required test functions
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Collect and display all compatibility tests
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in compatibility tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py
```

**Accept when:**
- test/test_compatibility.py exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions
- All compatibility tests pass in CI pipeline before deployment
- New public API endpoints include corresponding compatibility tests before merge approval

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. CI pipeline MUST fail if compatibility tests are missing for new public API endpoints. Pull requests adding or modifying public APIs MUST require compatibility test updates before merge approval. Quarterly audits of API endpoints MUST ensure compatibility test coverage is complete.
</enforcement>
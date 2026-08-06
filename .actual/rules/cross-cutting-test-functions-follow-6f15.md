# Establish HTTP Client Compatibility Testing for Public API Contracts: Test Functions Follow

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and HTTP protocol version compatibility tests in the codebase.

### Rules

- **R-COMPAT-001** SHOULD: Test functions SHOULD follow naming convention `test_<feature>` for discoverability and automated test execution.

### Verify

```bash
# Verify compatibility test functions exist and follow naming convention
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Collect and display all compatibility test functions
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py
```

**Accept when:**
- `test/test_compatibility.py` exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions (e.g., `test_h2_version_check`)
- All compatibility tests pass in CI pipeline before deployment
- Test functions follow the `test_<feature>` naming convention for automated discovery

<enforcement>
Claude Code MUST NOT skip or defer verification of compatibility test naming conventions and coverage. All new public API endpoints MUST include corresponding compatibility tests before merge approval.
</enforcement>
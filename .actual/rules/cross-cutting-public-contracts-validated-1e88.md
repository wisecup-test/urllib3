# Establish HTTP Client Compatibility Testing for Public API Contracts: Public Contracts Validated

These rules are ALWAYS ACTIVE for all public-facing HTTP API endpoints, HTTP client library integrations, and protocol version compatibility validation.

### Rules

- **R-COMPAT-001** MUST: All public API contracts MUST be validated through compatibility tests that exercise multiple HTTP client implementations (urllib, urllib3, http.cookiejar).

### Verify

```bash
# Verify compatibility test file exists and contains required test functions
grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py

# Verify test collection includes HTTP client compatibility tests
python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"

# Verify HTTP client library coverage in compatibility tests
grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py
```

**Accept when:**
- test/test_compatibility.py exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions
- All compatibility tests pass in CI pipeline before deployment

<enforcement>
Claude Code MUST NOT skip or defer verification of compatibility test presence and execution. Violations result in CI pipeline failure and pull request rejection until compatibility tests are added or updated.
</enforcement>
# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Fixtures Generating Tls

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-FIXTURES-TLS-001** MUST: Fixtures generating TLS certificates MUST coordinate with parameterized loopback_host values using trustme.CA.issue_cert()

### Verify

```bash
# Verify parameterized loopback_host fixture exists with all three address variants
grep -r '@pytest.fixture(params=' test/ | grep -E '(localhost|127\.0\.0\.1|::1)'

# Verify certificate generation fixtures coordinate with loopback_host parameter values
grep -r 'trustme\.CA' test/ | grep 'issue_cert'

# Verify connection state tracking uses set-based aggregation patterns
grep -r '\.add\(' test/ | grep -E '(connections|tls_versions)'

# Verify test collection shows parameterized test instances for each loopback address variant
pytest --collect-only test/ | grep -E '\[(localhost|127\.0\.0\.1|::1)\]'
```

**Accept when:**
- Parameterized loopback_host fixture exists in conftest.py with all three address variants (localhost, 127.0.0.1, ::1)
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (connections.add, tls_versions.add)

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations are detected via pytest --collect-only output inspection during CI, code review verification of new network tests, and grep-based static analysis checking for hardcoded loopback addresses. CI fails if new tests hardcode loopback addresses instead of using parameterized fixtures. Code review blocks PRs that bypass fixture-based server lifecycle management.
</enforcement>
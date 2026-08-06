# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Test Fixtures Accessing

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, including TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-FIXTURE-001** MUST: Test fixtures accessing network resources MUST use `pytest.fixture` with `params` to parameterize loopback hosts across `localhost`, `127.0.0.1`, and `::1`.

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
- Parameterized `loopback_host` fixture exists in `conftest.py` with all three address variants (`localhost`, `127.0.0.1`, `::1`)
- Certificate generation fixtures coordinate with `loopback_host` parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (`connections.add()`, `tls_versions.add()`)
- New network tests use `loopback_host` fixture parameter instead of hardcoded addresses

<enforcement>
Claude Code MUST NOT skip or defer verification. All network-bound test fixtures MUST be parameterized across loopback address variants. Violations detected during code review or CI inspection MUST block PR merge until remediated or formally excepted with documented rationale.
</enforcement>

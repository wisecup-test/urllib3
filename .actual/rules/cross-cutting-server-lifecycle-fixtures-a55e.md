# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Server Lifecycle Fixtures

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-FIXTURE-001** MUST: Server lifecycle fixtures MUST yield ServerConfig objects through context managers (run_server_in_thread, run_server_and_proxy_in_thread).
- **R-FIXTURE-002** MUST: Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization across IPv4 and IPv6 loopback addresses.
- **R-FIXTURE-003** MUST: Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects.
- **R-FIXTURE-004** MUST: Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only certificates.
- **R-FIXTURE-005** MUST: Wrap server lifecycle in context managers that yield ServerConfig and ensure cleanup on fixture teardown.
- **R-FIXTURE-006** MUST: Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests.
- **R-FIXTURE-007** SHOULD: Use tmp_path_factory.mktemp() for certificate storage to ensure test isolation and automatic cleanup.
- **R-FIXTURE-008** SHOULD: Configure pytest to display parameter values in test names using --verbose flag and ensure CI captures full pytest output.
- **R-FIXTURE-009** SHOULD: Implement explicit cleanup in fixture finalizers and use pytest-timeout to detect hanging server threads.

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

# Verify no hardcoded loopback addresses in test files
grep -r '127\.0\.0\.1\|localhost\|::1' test/ | grep -v '@pytest.fixture' | grep -v 'params=' || echo 'No hardcoded addresses found'
```

**Accept when:**
- Parameterized loopback_host fixture exists in conftest.py with all three address variants (localhost, 127.0.0.1, ::1)
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (connections.add, tls_versions.add)
- Server lifecycle fixtures yield ServerConfig objects through context managers
- HAS_IPV6 detection is implemented with appropriate pytest.skip() calls
- No hardcoded loopback addresses appear in test functions

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations detected by grep-based static analysis or pytest collection inspection MUST be flagged during code review. New network tests MUST use loopback_host fixture parameterization. Non-parameterized network tests require documented exception approval from test infrastructure maintainer with # noqa comments and issue tracker references.
</enforcement>
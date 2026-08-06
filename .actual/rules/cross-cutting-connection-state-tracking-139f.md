# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Connection State Tracking

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-PYTEST-001** MUST: Connection state tracking MUST use set-based aggregation patterns (connections.add(conn), tls_versions.add(_sock.version())).
- **R-PYTEST-002** MUST: Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization across IPv4 and IPv6 loopback addresses.
- **R-PYTEST-003** MUST: Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only certificates.
- **R-PYTEST-004** MUST: Wrap server lifecycle in context managers (run_server_in_thread) that yield ServerConfig and ensure cleanup on fixture teardown.
- **R-PYTEST-005** SHOULD: Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects.
- **R-PYTEST-006** SHOULD: Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests.
- **R-PYTEST-007** SHOULD: Use tmp_path_factory.mktemp() for certificate storage to ensure test isolation and automatic cleanup.
- **R-PYTEST-008** MUST: New network tests MUST use loopback_host fixture parameter instead of hardcoding loopback addresses.

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
grep -r '127\.0\.0\.1\|localhost\|::1' test/ | grep -v '@pytest.fixture' | grep -v 'params=' | grep -v '#' || echo 'No hardcoded addresses found'
```

**Accept when:**
- Parameterized loopback_host fixture exists in conftest.py with all three address variants (localhost, 127.0.0.1, ::1)
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (connections.add, tls_versions.add)
- No new tests hardcode loopback addresses; all network tests use the parameterized fixture
- Server lifecycle management uses context managers with proper cleanup on fixture teardown

<enforcement>
Claude Code MUST NOT skip or defer verification. All network-bound test code MUST be inspected for compliance with R-PYTEST-001 through R-PYTEST-008. Code review MUST block PRs that bypass fixture-based server lifecycle management or hardcode loopback addresses. CI MUST fail if pytest --collect-only does not show parameterized test instances for each loopback address variant.
</enforcement>
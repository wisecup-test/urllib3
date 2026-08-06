# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Ipv6 Tests Skip

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, including TLS/SSL certificate generation and validation tests, connection pooling scenarios, and socket-level test infrastructure in conftest.py and test files.

### Rules

- **R-IPV6-001** MUST: IPv6 tests MUST skip execution when HAS_IPV6 flag is false using pytest.skip().
- **R-IPV6-002** MUST: Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization across IPv4 and IPv6 variants.
- **R-IPV6-003** MUST: Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only certificates.
- **R-IPV6-004** MUST: Wrap server lifecycle in context managers (run_server_in_thread) that yield ServerConfig and ensure cleanup on fixture teardown.
- **R-IPV6-005** MUST: Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests.
- **R-IPV6-006** SHOULD: Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects.
- **R-IPV6-007** SHOULD: Use tmp_path_factory.mktemp() for certificate storage to ensure test isolation and automatic cleanup.
- **R-IPV6-008** SHOULD: Use set-based aggregation (connections.add, tls_versions.add) for efficient state tracking across parameterized test executions.

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
- Connection state tracking uses set-based aggregation patterns
- IPv6 tests skip gracefully when HAS_IPV6 flag is false
- New network tests use loopback_host fixture parameter instead of hardcoded addresses

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-IPV6-### rules marked MUST are non-negotiable. Code review MUST block PRs that bypass fixture-based server lifecycle management or hardcode loopback addresses. CI MUST fail if new tests hardcode loopback addresses instead of using parameterized fixtures.
</enforcement>
# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Temporary Certificate Storage

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-PYTEST-FIXTURE-001** SHOULD: Temporary certificate storage SHOULD use pytest.TempPathFactory.mktemp() for isolated test execution.
- **R-PYTEST-FIXTURE-002** MUST: Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization across IPv4 and IPv6 variants.
- **R-PYTEST-FIXTURE-003** MUST: Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects.
- **R-PYTEST-FIXTURE-004** MUST: Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only certificates.
- **R-PYTEST-FIXTURE-005** MUST: Wrap server lifecycle in context managers (run_server_in_thread) that yield ServerConfig and ensure cleanup on fixture teardown.
- **R-PYTEST-FIXTURE-006** MUST: Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests.
- **R-PYTEST-FIXTURE-007** SHOULD: Use set-based aggregation (connections.add, tls_versions.add) for efficient state tracking across parameterized test executions.
- **R-PYTEST-FIXTURE-008** MUST: Reject hardcoded loopback addresses in test files; all network tests MUST use parameterized loopback_host fixture.
- **R-PYTEST-FIXTURE-009** MUST: Implement explicit cleanup in fixture finalizers and use pytest-timeout to detect hanging server threads.

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
grep -r '127\.0\.0\.1\|localhost\|::1' test/ | grep -v '@pytest.fixture' | grep -v 'loopback_host' || echo 'No hardcoded addresses found'

# Verify tmp_path_factory usage for certificate storage
grep -r 'tmp_path_factory\.mktemp' test/
```

**Accept when:**
- Parameterized loopback_host fixture exists in conftest.py with all three address variants (localhost, 127.0.0.1, ::1)
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (connections.add, tls_versions.add)
- No hardcoded loopback addresses appear in test files outside fixture definitions
- tmp_path_factory.mktemp() is used for certificate storage to ensure test isolation
- HAS_IPV6 detection is implemented with pytest.skip() for IPv6-dependent tests
- Server lifecycle is wrapped in context managers with explicit cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PYTEST-FIXTURE rules marked MUST are mandatory and MUST be verified before accepting test infrastructure changes. Violations MUST be caught during code review and CI inspection of pytest --collect-only output.
</enforcement>
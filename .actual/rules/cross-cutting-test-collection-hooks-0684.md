# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Test Collection Hooks

These rules are ALWAYS ACTIVE for pytest-based test suites with network resource access, TLS/SSL certificate generation and validation tests, connection pooling and socket-level test scenarios, and fixtures in conftest.py establishing test infrastructure.

### Rules

- **R-PYTEST-FIXTURE-001** SHOULD: Test collection hooks SHOULD implement pytest_addoption and pytest_collection_modifyitems to establish public configuration contracts.
- **R-PYTEST-FIXTURE-002** MUST: Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization across IPv4 and IPv6 loopback addresses.
- **R-PYTEST-FIXTURE-003** MUST: Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects.
- **R-PYTEST-FIXTURE-004** MUST: Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only certificates.
- **R-PYTEST-FIXTURE-005** MUST: Wrap server lifecycle in context managers (run_server_in_thread) that yield ServerConfig and ensure cleanup on fixture teardown.
- **R-PYTEST-FIXTURE-006** MUST: Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests.
- **R-PYTEST-FIXTURE-007** MUST: Use tmp_path_factory.mktemp() for certificate storage to ensure test isolation and automatic cleanup.
- **R-PYTEST-FIXTURE-008** MUST: Use set-based aggregation (connections.add, tls_versions.add) for efficient state tracking across parameterized test executions.
- **R-PYTEST-FIXTURE-009** MUST NOT: Hardcode single loopback addresses (127.0.0.1, localhost, ::1) in test files; use parameterized loopback_host fixture instead.
- **R-PYTEST-FIXTURE-010** MUST NOT: Duplicate test functions for each loopback address variant; leverage pytest parameterization to avoid code duplication.

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
grep -r '127\.0\.0\.1\|localhost\|::1' test/ | grep -v '@pytest.fixture' | grep -v 'loopback_host' | grep -v 'params=' || echo 'No hardcoded addresses found'
```

**Accept when:**
- Parameterized loopback_host fixture exists in conftest.py with all three address variants (localhost, 127.0.0.1, ::1)
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns (connections.add, tls_versions.add)
- No hardcoded loopback addresses appear in test files outside fixture definitions
- HAS_IPV6 detection is implemented with pytest.skip() for IPv6-dependent tests
- Server lifecycle is wrapped in context managers with proper cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST execute successfully before accepting changes to network-bound test infrastructure. Code review MUST block PRs that bypass fixture-based server lifecycle management or hardcode loopback addresses. CI MUST fail if new tests hardcode loopback addresses instead of using parameterized fixtures.
</enforcement>
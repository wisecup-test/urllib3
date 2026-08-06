# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Tests Skip Ipv6

These rules are ALWAYS ACTIVE for pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-TLS-001** MAY: Tests MAY skip IPv6 scenarios when HAS_IPV6 flag indicates platform lacks IPv6 support.
- **R-TLS-002** MUST: Use pytest.fixture with params=['localhost', '127.0.0.1', '::1'] for parameterized loopback host testing across IPv4 and IPv6 addresses.
- **R-TLS-003** MUST: Implement trustme CA infrastructure for dynamic server fixtures with both SAN and non-SAN certificates.
- **R-TLS-004** MUST: Track connection object lifecycle and reuse patterns through observable collections (tls_versions set, connections set) initialized within fixture scope.
- **R-TLS-005** MUST: Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions.
- **R-TLS-006** MUST: Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation.
- **R-TLS-007** MUST: Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests.
- **R-TLS-008** MUST: Ensure each test creates fresh connection pool instances and verify fixture scope prevents state leakage across tests.

### Verify

```bash
# Verify parameterized fixtures with loopback host variants
grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/

# Verify trustme CA usage
grep -r 'trustme\.CA\(\)' test/

# Verify TLS version detection pattern
grep -r 'tls_versions\.add\(_sock\.version\(\)\)' test/

# Verify connection tracking pattern
grep -r 'connections\.add\(conn\)' test/

# Verify HAS_IPV6 flag implementation
grep -r 'HAS_IPV6' test/

# Verify context manager server lifecycle
grep -r 'run_server_in_thread' test/
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns.
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms.
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns.
- HAS_IPV6 platform detection is implemented and IPv6 tests skip gracefully on platforms without IPv6 support.
- Server lifecycle is properly managed through context managers with guaranteed cleanup.
- Certificate directories are isolated per test fixture invocation using pytest.TempPathFactory.mktemp().

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-TLS rules are mandatory for test infrastructure compliance. Violations detected during CI pipeline execution or code review must block merge.
</enforcement>
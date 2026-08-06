# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Tls Test Servers

These rules are ALWAYS ACTIVE for pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-TLS-001** MUST: TLS test servers MUST use trustme CA for certificate generation with both SAN and non-SAN certificate variants.
- **R-TLS-002** MUST: Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation.
- **R-TLS-003** MUST: Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests.
- **R-TLS-004** MUST: Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions.
- **R-TLS-005** MUST: Initialize observable collections (tls_versions set, connections set) within fixture scope to ensure test isolation and prevent cross-test contamination.
- **R-TLS-006** SHOULD: Use parameterized fixtures (@pytest.fixture(params=['localhost', '127.0.0.1', '::1'])) to systematically test network protocol variants across IPv4 and IPv6.
- **R-TLS-007** SHOULD: Implement connection tracking through data access patterns (connections.add(conn)) to enable verification of connection reuse and lifecycle behavior.

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
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns
- All TLS test fixtures properly clean up server resources and certificate directories
- New TLS tests follow parameterization and context manager patterns established in existing fixtures

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All TLS test infrastructure changes MUST be validated against R-TLS-001 through R-TLS-007 before acceptance. CI pipeline failures related to test fixture cleanup or trustme CA integration MUST be resolved before merge.
</enforcement>
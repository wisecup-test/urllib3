# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Connection Pool Tests

These rules are ALWAYS ACTIVE for pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-POOL-001** MUST: Connection pool tests MUST track connection object identity through data access patterns that add connections to observable collections.
- **R-POOL-002** MUST: Use pytest fixtures with parameterization across loopback host variants (localhost, 127.0.0.1, ::1) to validate network protocol compatibility.
- **R-POOL-003** MUST: Implement TLS certificate generation and validation testing using trustme CA infrastructure with both SAN and non-SAN certificates.
- **R-POOL-004** MUST: Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions.
- **R-POOL-005** MUST: Initialize observable collections (tls_versions set, connections set) within fixture scope to ensure test isolation and prevent cross-test contamination.
- **R-POOL-006** SHOULD: Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation.
- **R-POOL-007** SHOULD: Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests.
- **R-POOL-008** SHOULD: Document minimum supported OpenSSL and Python versions, and add version detection to test setup validation.

### Verify

```bash
# Verify parameterized fixtures with loopback host variants
grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/

# Verify trustme CA usage
grep -r 'trustme\.CA\(\)' test/

# Verify TLS version tracking pattern
grep -r 'tls_versions\.add\(_sock\.version\(\)\)' test/

# Verify connection object tracking pattern
grep -r 'connections\.add\(conn\)' test/
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns
- Server lifecycle is wrapped in context managers ensuring resource cleanup
- Observable collections are initialized within fixture scope with no cross-test contamination

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-POOL rules are mandatory for connection pool test infrastructure. Violations block merge and trigger CI build failures.
</enforcement>
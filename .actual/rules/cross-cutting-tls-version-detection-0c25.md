# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Tls Version Detection

These rules are ALWAYS ACTIVE for pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-TLS-001** MUST: TLS version detection MUST populate version sets through socket-level inspection using `_sock.version()` calls.
- **R-TLS-002** MUST: Use pytest fixtures with parameterization across loopback host variants (localhost, 127.0.0.1, ::1) for comprehensive network protocol coverage.
- **R-TLS-003** MUST: Implement dynamic certificate generation using trustme CA infrastructure for both SAN and non-SAN certificate configurations.
- **R-TLS-004** MUST: Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions.
- **R-TLS-005** MUST: Initialize observable collections (tls_versions set, connections set) within fixture scope to ensure test isolation and prevent cross-test contamination.
- **R-TLS-006** SHOULD: Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation.
- **R-TLS-007** SHOULD: Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests.
- **R-TLS-008** SHOULD: Document minimum supported OpenSSL and Python versions, and add version detection to test setup validation.

### Verify

```bash
# Verify parameterized fixtures with loopback host variants
grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/

# Verify trustme CA usage for certificate generation
grep -r 'trustme\.CA\(\)' test/

# Verify TLS version detection pattern
grep -r 'tls_versions\.add\(_sock\.version\(\)\)' test/

# Verify connection tracking pattern
grep -r 'connections\.add\(conn\)' test/
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns.
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms.
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns.
- Server lifecycle management uses context managers with proper exception handling and resource cleanup.
- Observable collections are initialized within fixture scope with no cross-test state leakage.

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-TLS rules marked MUST are non-negotiable. Code review MUST verify new test fixtures follow parameterization and context manager patterns. CI pipeline MUST execute pytest test suite with coverage reporting and fail if test fixtures do not properly clean up server resources.
</enforcement>
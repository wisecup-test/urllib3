# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Test Fixtures Parameterize

These rules are ALWAYS ACTIVE for all pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-TLS-001** MUST: Test fixtures MUST parameterize loopback host testing across localhost, 127.0.0.1, and ::1 to validate IPv4 and IPv6 compatibility.
- **R-TLS-002** MUST: Use trustme CA infrastructure for dynamic server fixtures with both SAN and non-SAN certificates instead of pre-generated static certificates.
- **R-TLS-003** MUST: Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions.
- **R-TLS-004** MUST: Initialize observable collections (tls_versions set, connections set) within fixture scope to ensure test isolation and prevent cross-test contamination.
- **R-TLS-005** MUST: Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation.
- **R-TLS-006** SHOULD: Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests.
- **R-TLS-007** SHOULD: Document minimum supported OpenSSL and Python versions and add version detection to test setup validation.
- **R-TLS-008** SHOULD: Monitor IPv6 test skip rates to detect platform configuration drift and ensure CI environments enable IPv6 support.

### Verify

```bash
# Verify parameterized fixtures across loopback hosts
grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/

# Verify trustme CA usage
grep -r 'trustme\.CA\(\)' test/

# Verify TLS version detection pattern
grep -r 'tls_versions\.add\(_sock\.version\(\)\)' test/

# Verify connection tracking pattern
grep -r 'connections\.add\(conn\)' test/

# Verify context manager server lifecycle
grep -r 'run_server_in_thread' test/

# Verify TempPathFactory usage
grep -r 'TempPathFactory\.mktemp' test/
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns.
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms.
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns.
- Server lifecycle is wrapped in context managers ensuring resource cleanup on test failure.
- Each test creates fresh connection pool instances with isolated certificate directories.
- HAS_IPV6 platform detection is implemented with clear skip messages for unavailable IPv6 environments.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-TLS-### rules marked MUST are non-negotiable for test fixture implementation. Code review MUST block merge if new TLS tests bypass trustme CA infrastructure or fail to parameterize loopback hosts. CI build MUST fail if test fixtures do not properly clean up server resources or if tests fail on IPv6-enabled platforms without documented justification.
</enforcement>
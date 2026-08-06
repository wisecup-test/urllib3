# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Test Fixtures Use

These rules are ALWAYS ACTIVE for all pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, and network protocol compatibility validation across IPv4 and IPv6.

### Rules

- **R-TLS-FIXTURES-001** SHOULD: Test fixtures SHOULD use pytest.TempPathFactory for certificate storage to ensure test isolation and cleanup.

### Verify

```bash
# Verify parameterized fixtures with loopback host variants
grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/

# Verify trustme CA integration
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
- pytest.TempPathFactory.mktemp() is used to create isolated certificate directories for each test fixture invocation
- HAS_IPV6 platform detection is implemented early in test suite initialization with clear skip messages
- Server lifecycle is wrapped in context managers (run_server_in_thread) to guarantee cleanup
- Observable collections (tls_versions set, connections set) are initialized within fixture scope to prevent cross-test contamination

<enforcement>
Clause Code MUST NOT skip or defer verification. All grep commands MUST return matches. Test suite MUST execute successfully across parameterized variants. Connection pool state tracking MUST be observable through collection-based patterns.
</enforcement>
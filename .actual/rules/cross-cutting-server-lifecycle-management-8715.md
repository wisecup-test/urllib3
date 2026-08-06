# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Server Lifecycle Management

These rules are ALWAYS ACTIVE for pytest-based test suites requiring TLS server fixtures, connection pool manager test scenarios, network protocol compatibility validation, and certificate validation testing.

### Rules

- **R-LIFECYCLE-001** SHOULD: Server lifecycle management SHOULD use context managers (run_server_in_thread) to guarantee resource cleanup.

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

# Verify context manager usage for server lifecycle
grep -r 'run_server_in_thread' test/
```

**Accept when:**
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns
- Server resources are properly cleaned up even when tests fail or raise exceptions
- Each test creates fresh connection pool instances with no state leakage between tests

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST execute pytest test suite with coverage reporting. Code review MUST verify new test fixtures follow parameterization and context manager patterns. CI build MUST fail if test fixtures do not properly clean up server resources. Code review MUST block merge if new TLS tests bypass trustme CA infrastructure.
</enforcement>
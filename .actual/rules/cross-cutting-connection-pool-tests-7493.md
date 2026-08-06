# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Connection Pool Tests

These rules are ALWAYS ACTIVE for all test modules validating HTTP/HTTPS connection pooling behavior, TLS certificate validation, PoolManager lifecycle, and data access patterns with network I/O.

### Rules

- **R-FIXTURE-001** SHOULD: Connection pool tests SHOULD use fixtures that yield ServerConfig objects containing all necessary connection parameters.

### Verify

```bash
# Verify pytest fixtures are defined for network infrastructure
grep -r '@pytest.fixture' test/ | grep -E '(loopback_host|san_server|no_san_server)' | wc -l

# Verify server lifecycle fixtures use contextmanager pattern
grep -r 'run_server_in_thread' test/ | grep 'with.*as' | wc -l

# Verify parameterized fixtures generate multiple test variants
pytest test/test_poolmanager.py -v --collect-only | grep -E '(localhost|127.0.0.1|::1)' | wc -l
```

**Accept when:**
- At least 3 pytest fixtures are defined for network infrastructure setup (loopback_host, san_server, no_san_server)
- Server lifecycle fixtures use contextmanager pattern (with statement) for resource cleanup
- Parameterized fixtures generate multiple test variants for protocol coverage (IPv4, IPv6)

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations are caught by automated pytest collection analysis in CI pipeline, code review checklist enforcement, and static analysis scanning for anti-patterns. New test files using unittest setUp/tearDown instead of pytest fixtures will cause CI pipeline failure. Code review blocks merge if data access tests lack proper fixture-based resource cleanup. Approved exceptions require pull request comment justification and Test Infrastructure Team review within 2 business days.
</enforcement>
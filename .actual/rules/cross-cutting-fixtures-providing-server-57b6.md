# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Fixtures Providing Server

These rules are ALWAYS ACTIVE for all test modules validating HTTP/HTTPS connection pooling behavior, TLS certificate validation, integration tests for PoolManager and connection lifecycle, and tests exercising data access patterns with network I/O.

### Rules

- **R-FIXTURE-001** MUST: Fixtures providing server configurations MUST use contextmanager patterns to ensure proper resource cleanup and teardown.

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
Claude Code MUST NOT skip or defer verification. All new test files in test/ directory MUST use pytest fixtures with contextmanager-based resource cleanup for data access layer testing. Code review MUST block merge if data access tests lack proper fixture-based resource cleanup. CI pipeline MUST fail if new test files use unittest setUp/tearDown instead of pytest fixtures.
</enforcement>
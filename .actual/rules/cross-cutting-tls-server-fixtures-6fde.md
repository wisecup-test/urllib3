# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Tls Server Fixtures

These rules are ALWAYS ACTIVE for all test modules validating HTTP/HTTPS connection pooling behavior, TLS certificate validation, integration tests for PoolManager and connection lifecycle, and tests exercising data access patterns with network I/O.

### Rules

- **R-TLS-001** SHOULD: TLS server fixtures SHOULD generate certificates dynamically using trustme or equivalent to avoid hardcoded test credentials.
- **R-TLS-002** MUST: Place shared fixtures in test/conftest.py to make them available across all test modules without explicit imports.
- **R-TLS-003** MUST: Use @pytest.fixture(params=[...]) for protocol variants and typing.Generator return types for contextmanager-based fixtures.
- **R-TLS-004** MUST: Implement server fixtures using 'with run_server_in_thread(...)' pattern to ensure cleanup via context manager __exit__.
- **R-TLS-005** MUST: Add HAS_IPV6 detection and pytest.skip() guards for IPv6-dependent tests to prevent failures on limited CI environments.
- **R-TLS-006** MUST: Use tmp_path_factory.mktemp() for certificate storage to leverage pytest's automatic temporary directory cleanup.

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
Claude Code MUST NOT skip or defer verification. CI pipeline fails if new test files in test/ directory use unittest setUp/tearDown instead of pytest fixtures. Code review blocks merge if data access tests lack proper fixture-based resource cleanup. Automated linting warns on direct socket/server instantiation in test functions without fixture injection.
</enforcement>
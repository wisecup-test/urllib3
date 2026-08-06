# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Parameterized Fixtures Used

These rules are ALWAYS ACTIVE for all test modules validating HTTP/HTTPS connection pooling behavior, TLS certificate validation, integration tests for PoolManager and connection lifecycle, and tests exercising data access patterns with network I/O.

### Rules

- **R-FIXTURE-001** MUST: Parameterized fixtures MUST be used to test data access patterns across multiple protocol variants (IPv4, IPv6, loopback addresses).
- **R-FIXTURE-002** MUST: Shared fixtures MUST be placed in test/conftest.py to make them available across all test modules without explicit imports.
- **R-FIXTURE-003** MUST: Server lifecycle fixtures MUST use contextmanager pattern (with statement) for resource cleanup to ensure proper teardown even when tests fail.
- **R-FIXTURE-004** MUST: Parameterized fixtures MUST use @pytest.fixture(params=[...]) decorator with typing.Generator return types for contextmanager-based fixtures.
- **R-FIXTURE-005** MUST: IPv6-dependent tests MUST implement HAS_IPV6 detection and pytest.skip() guards to prevent failures on CI environments without IPv6 support.
- **R-FIXTURE-006** MUST: Certificate storage MUST use tmp_path_factory.mktemp() to leverage pytest's automatic temporary directory cleanup.
- **R-FIXTURE-007** SHOULD: New data access tests SHOULD use fixture-based resource setup rather than unittest setUp/tearDown methods.
- **R-FIXTURE-008** SHOULD: Test modules SHOULD include fixture-specific logging and health checks to aid debugging of fixture setup failures.

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
Claude Code MUST NOT skip or defer verification. Violations are detected by automated pytest collection analysis in CI pipeline, code review checklist enforcement, and static analysis scanning for anti-patterns. CI pipeline fails if new test files use unittest setUp/tearDown instead of pytest fixtures. Code review blocks merge if data access tests lack proper fixture-based resource cleanup. Exceptions require pull request comment justification and Test Infrastructure Team review within 2 business days.
</enforcement>
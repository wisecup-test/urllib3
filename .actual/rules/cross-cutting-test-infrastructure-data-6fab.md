# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Test Infrastructure Data

These rules are ALWAYS ACTIVE for all test files in the `test/` directory that validate data access patterns, HTTP/HTTPS connection pooling behavior, TLS certificate validation, and integration tests for PoolManager and connection lifecycle.

### Rules

- **R-PYTEST-FIXTURE-001** MUST: Test infrastructure for data access patterns MUST use pytest fixtures to manage lifecycle of network resources, servers, and connection pools.
- **R-PYTEST-FIXTURE-002** MUST: Shared fixtures MUST be placed in `test/conftest.py` to make them available across all test modules without explicit imports.
- **R-PYTEST-FIXTURE-003** MUST: Server lifecycle fixtures MUST use the `with run_server_in_thread(...)` pattern to ensure cleanup via context manager `__exit__`.
- **R-PYTEST-FIXTURE-004** MUST: Parameterized fixtures for protocol variants MUST use `@pytest.fixture(params=[...])` with `typing.Generator` return types for contextmanager-based fixtures.
- **R-PYTEST-FIXTURE-005** MUST: IPv6-dependent tests MUST implement `HAS_IPV6` detection and use `pytest.skip()` guards to prevent failures on limited CI environments.
- **R-PYTEST-FIXTURE-006** MUST: Certificate storage MUST use `tmp_path_factory.mktemp()` to leverage pytest's automatic temporary directory cleanup.
- **R-PYTEST-FIXTURE-007** SHOULD: New data access tests SHOULD use fixture-based resource cleanup rather than unittest `setUp`/`tearDown` methods.
- **R-PYTEST-FIXTURE-008** SHOULD: Test modules SHOULD document approved exceptions to fixture-based setup in module docstrings with expiration dates and migration plans.

### Verify

```bash
# Verify at least 3 pytest fixtures are defined for network infrastructure setup
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
- No new test files in `test/` directory use unittest `setUp`/`tearDown` instead of pytest fixtures
- All data access tests include proper fixture-based resource cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. Violations are caught by CI pipeline automated pytest collection analysis, code review checklist enforcement, and static analysis scanning for anti-patterns. Approved exceptions require pull request comment justification and Test Infrastructure Team review within 2 business days.
</enforcement>
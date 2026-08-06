# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Tests Skip Execution

These rules are ALWAYS ACTIVE for all test files in the `test/` directory that validate HTTP/HTTPS connection pooling behavior, TLS certificate validation, PoolManager lifecycle, and data access patterns with network I/O.

### Rules

- **R-PYTEST-001** MUST: Use pytest fixtures (loopback_host, san_server, no_san_server) for network infrastructure setup in all data access layer tests.
- **R-PYTEST-002** MUST: Implement server lifecycle fixtures using the `with run_server_in_thread(...)` contextmanager pattern to ensure resource cleanup via `__exit__`.
- **R-PYTEST-003** MUST: Place shared fixtures in `test/conftest.py` to make them available across all test modules without explicit imports.
- **R-PYTEST-004** SHOULD: Use `@pytest.fixture(params=[...])` for protocol variants (IPv4, IPv6, different TLS configurations) to enable comprehensive validation without code duplication.
- **R-PYTEST-005** MAY: Tests MAY skip execution when required network capabilities (e.g., IPv6) are unavailable on the test host using `pytest.skip()` with HAS_IPV6 detection.
- **R-PYTEST-006** MUST: Add HAS_IPV6 detection and `pytest.skip()` guards for IPv6-dependent tests to prevent failures on limited CI environments.
- **R-PYTEST-007** SHOULD: Use `tmp_path_factory.mktemp()` for certificate storage to leverage pytest's automatic temporary directory cleanup.
- **R-PYTEST-008** MUST NOT: Use unittest `setUp`/`tearDown` methods for test infrastructure initialization in data access layer tests; use pytest fixtures instead.
- **R-PYTEST-009** MUST NOT: Create module-level or session-level server instances shared across all tests; maintain test isolation through fixture scoping.

### Verify

```bash
# Verify pytest fixtures are defined for network infrastructure
grep -r '@pytest.fixture' test/ | grep -E '(loopback_host|san_server|no_san_server)' | wc -l

# Verify server lifecycle fixtures use contextmanager pattern
grep -r 'run_server_in_thread' test/ | grep 'with.*as' | wc -l

# Verify parameterized fixtures generate multiple test variants
pytest test/test_poolmanager.py -v --collect-only | grep -E '(localhost|127.0.0.1|::1)' | wc -l

# Verify no unittest setUp/tearDown patterns in test files
grep -r 'def setUp\|def tearDown' test/ | grep -v '.pyc' | wc -l
```

**Accept when:**
- At least 3 pytest fixtures are defined for network infrastructure setup (loopback_host, san_server, no_san_server)
- Server lifecycle fixtures use contextmanager pattern (with statement) for resource cleanup
- Parameterized fixtures generate multiple test variants for protocol coverage (IPv4, IPv6)
- No unittest setUp/tearDown patterns are found in test files

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new test files in test/ directory use unittest setUp/tearDown instead of pytest fixtures. Code review MUST block merge if data access tests lack proper fixture-based resource cleanup. Automated linting MUST warn on direct socket/server instantiation in test functions without fixture injection.
</enforcement>
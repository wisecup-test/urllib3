# Standardize Public API Contract Testing with Pytest-Based Test Functions: Test Modules Include

These rules are ALWAYS ACTIVE for all test modules in the `test/` directory that validate public API contracts for urllib3 core modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-TEST-001** MUST: Test modules include test functions following the `test_*` naming convention using pytest as the test framework.
- **R-TEST-002** MUST: Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost, exceptions) have corresponding test modules with dedicated contract validation functions.
- **R-TEST-003** MUST: Test modules validate public interfaces including proxy management, file posting, connection handling, and HTTP/2 operations at appropriate abstraction layers (unit, integration, socket-level).
- **R-TEST-004** SHOULD: Test modules use conftest.py for shared fixtures including server configuration, certificate generation (trustme.CA), and loopback host parameterization.
- **R-TEST-005** SHOULD: Test modules organize by functional domain matching source module structure (e.g., test_connectionpool.py tests src/urllib3/connectionpool.py).
- **R-TEST-006** MAY: Test modules MAY include specialized test infrastructure such as mock servers, socket handlers, and certificate authorities (trustme.CA) to simulate external dependencies.
- **R-TEST-007** MAY: Test modules MAY use pytest.raises context manager for exception contract validation and pytest.skip for platform-specific or environment-dependent test exclusion.

### Verify

```bash
# Count test functions following test_* naming convention
grep -r "^def test_" test/ | wc -l

# Collect pytest test functions
pytest --collect-only -q test/ | grep "<Function" | wc -l

# Count pytest fixtures in conftest.py
grep -r "@pytest.fixture" test/conftest.py | wc -l

# Run contract tests for core modules
pytest test/ -v --tb=short -k "test_proxy or test_connection or test_filepost"
```

**Accept when:**
- All test modules in `test/` directory contain test functions following the `test_*` naming convention and use pytest as the test framework
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary
- Shared fixtures are defined in conftest.py including server configuration, certificate generation, and loopback host parameterization

<enforcement>
Claude Code MUST NOT skip or defer verification. All test modules MUST follow pytest conventions and include explicit contract validation for public APIs. CI pipeline pytest execution with mandatory test pass requirement before merge is required.
</enforcement>
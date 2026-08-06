# Standardize Public API Contract Testing with Pytest-Based Test Functions: Test Modules Organize

These rules are ALWAYS ACTIVE for all test modules in the `test/` directory that validate public API contracts for urllib3 core modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-TEST-001** MUST: Test modules MUST organize contract tests by functional domain (e.g., test_proxymanager.py, test_filepost.py, test_connectionpool.py, test_connection.py) with each module testing a cohesive set of related public interfaces.
- **R-TEST-002** MUST: All test functions MUST follow the `test_*` naming convention and use pytest as the test framework.
- **R-TEST-003** MUST: Public API contracts for HTTP client operations including request/response handling, connection pooling, proxy configuration, and SSL/TLS operations MUST have corresponding contract validation functions.
- **R-TEST-004** MUST: Test modules MUST use conftest.py for shared fixtures including server configuration, certificate generation (trustme.CA), and loopback host parameterization.
- **R-TEST-005** MUST: Socket-level integration tests MUST use handler functions (e.g., multicookie_response_handler, socket_handler) that simulate protocol-level interactions.
- **R-TEST-006** MUST: Exception contract validation MUST use pytest.raises context manager.
- **R-TEST-007** SHOULD: Platform-specific or environment-specific tests SHOULD use pytest.skip when required dependencies are unavailable.
- **R-TEST-008** MAY: Legacy test modules written before pytest adoption MAY use unittest.TestCase-based test classes with test module maintainer approval.

### Verify

```bash
# Count test functions following test_* naming convention
grep -r "^def test_" test/ | wc -l

# Collect pytest test functions
pytest --collect-only -q test/ | grep "<Function" | wc -l

# Count shared fixtures in conftest.py
grep -r "@pytest.fixture" test/conftest.py | wc -l

# Run contract tests for core modules
pytest test/ -v --tb=short -k "test_proxy or test_connection or test_filepost"
```

**Accept when:**
- All test modules in `test/` directory contain test functions following the `test_*` naming convention and use pytest as the test framework.
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions.
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary.
- conftest.py contains shared fixtures for server configuration, certificate generation, and loopback host parameterization.
- Socket-level integration tests use handler functions that simulate protocol-level interactions.
- Exception contracts are validated using pytest.raises context manager.

<enforcement>
Claude Code MUST NOT skip or defer verification. All test modules MUST be organized by functional domain, follow pytest conventions, and include contract validation for public APIs. CI pipeline pytest execution with mandatory test pass requirement before merge is required. Code review verification that new public APIs include corresponding contract tests is mandatory. Test coverage reporting with specific tracking of public API function coverage is mandatory.
</enforcement>
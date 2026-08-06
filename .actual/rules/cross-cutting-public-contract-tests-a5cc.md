# Standardize Public API Contract Testing with Pytest-Based Test Functions: Public Contract Tests

These rules are ALWAYS ACTIVE for all public API contract tests in the urllib3 test suite, particularly test modules validating public interfaces from core modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-CONTRACT-001** MUST: Public API contract tests MUST validate both successful execution paths and error conditions using pytest.raises for exception verification.
- **R-CONTRACT-002** MUST: All test modules in the test/ directory MUST contain test functions following the test_* naming convention and use pytest as the test framework.
- **R-CONTRACT-003** MUST: Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) MUST have corresponding test modules with dedicated contract validation functions.
- **R-CONTRACT-004** SHOULD: Organize test modules by functional domain matching the source module structure (e.g., test_connectionpool.py tests src/urllib3/connectionpool.py).
- **R-CONTRACT-005** SHOULD: Use conftest.py for shared fixtures including server configuration, certificate generation (trustme.CA), and loopback host parameterization.
- **R-CONTRACT-006** SHOULD: Implement socket-level integration tests using handler functions (e.g., multicookie_response_handler, socket_handler) that simulate protocol-level interactions.
- **R-CONTRACT-007** SHOULD: Follow test function naming convention test_<contract_being_validated> with descriptive names that document the specific API behavior under test.
- **R-CONTRACT-008** MAY: Legacy test modules written before pytest adoption may use unittest.TestCase-based test classes (EXC-001).
- **R-CONTRACT-009** MAY: Platform-specific or environment-specific tests may skip contract validation using pytest.skip when required dependencies are unavailable (EXC-002).

### Verify

```bash
# Count test functions following test_* naming convention
grep -r "^def test_" test/ | wc -l

# Collect pytest test functions
pytest --collect-only -q test/ | grep "<Function" | wc -l

# Count pytest fixtures in conftest.py
grep -r "@pytest.fixture" test/conftest.py | wc -l

# Run contract tests for key modules
pytest test/ -v --tb=short -k "test_proxy or test_connection or test_filepost"
```

**Accept when:**
- All test modules in test/ directory contain test functions following the test_* naming convention and use pytest as the test framework
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary
- Exception handling is validated using pytest.raises context manager for all public API error conditions

<enforcement>
Clause MUST NOT skip or defer verification. CI pipeline pytest execution with mandatory test pass requirement before merge is required. Code review verification that new public APIs include corresponding contract tests is mandatory. Test coverage reporting with specific tracking of public API function coverage is mandatory.
</enforcement>
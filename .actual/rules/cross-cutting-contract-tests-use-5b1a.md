# Standardize Public API Contract Testing with Pytest-Based Test Functions: Contract Tests Use

These rules are ALWAYS ACTIVE for all public API contract tests in the urllib3 codebase, particularly test modules validating public interfaces from core modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-CONTRACT-001** SHOULD: Contract tests SHOULD use pytest fixtures for test configuration, dependency injection, and resource management (e.g., @pytest.fixture for server setup, certificate generation, and loopback host configuration).
- **R-CONTRACT-002** MUST: All test modules in the test/ directory MUST contain test functions following the test_* naming convention and use pytest as the test framework.
- **R-CONTRACT-003** MUST: Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) MUST have corresponding test modules with dedicated contract validation functions.
- **R-CONTRACT-004** SHOULD: Test modules SHOULD be organized by functional domain matching the source module structure (e.g., test_connectionpool.py tests src/urllib3/connectionpool.py).
- **R-CONTRACT-005** SHOULD: Socket-level integration tests SHOULD use handler functions (e.g., multicookie_response_handler, socket_handler) that simulate protocol-level interactions.
- **R-CONTRACT-006** SHOULD: Test function naming SHOULD follow the convention test_<contract_being_validated> with descriptive names that document the specific API behavior under test.
- **R-CONTRACT-007** SHOULD: Exception contract validation SHOULD leverage pytest.raises context manager for exception type and message validation.
- **R-CONTRACT-008** SHOULD: Platform-specific or environment-dependent tests SHOULD use pytest.skip when required dependencies are unavailable.
- **R-CONTRACT-009** MAY: Legacy test modules written before pytest adoption MAY use unittest.TestCase-based test classes with test module maintainer approval.

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
- conftest.py contains shared fixtures for server configuration, certificate generation, and loopback host parameterization
- New public APIs include corresponding contract tests in the same commit

<enforcement>
Clause MUST NOT skip or defer verification. CI pipeline pytest execution with mandatory test pass requirement before merge is required. Code review verification that new public APIs include corresponding contract tests is mandatory. Test coverage reporting with specific tracking of public API function coverage is mandatory. Violation handling includes CI build failure when contract tests fail or are missing for modified public APIs, code review rejection for pull requests adding or modifying public APIs without corresponding test updates, and automated coverage reports flagging public API functions lacking contract test coverage.
</enforcement>
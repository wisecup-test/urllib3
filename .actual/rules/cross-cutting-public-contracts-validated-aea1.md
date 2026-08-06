# Standardize Public API Contract Testing with Pytest-Based Test Functions: Public Contracts Validated

These rules are ALWAYS ACTIVE for all public API contracts and test modules in the urllib3 codebase, particularly test files validating public interfaces through pytest-based test functions.

### Rules

- **R-CONTRACT-001** MUST: All public API contracts MUST be validated through dedicated test functions following the `test_*` naming convention and using pytest as the test framework.
- **R-CONTRACT-002** MUST: All public functions, methods, and classes exported from urllib3 modules (poolmanager, connectionpool, connection, fields, filepost, exceptions) MUST have corresponding contract validation tests.
- **R-CONTRACT-003** MUST: Public API contracts for HTTP client operations including request/response handling, connection pooling, proxy configuration, and SSL/TLS operations MUST be validated through dedicated test functions.
- **R-CONTRACT-004** MUST: Integration points between urllib3 and external systems including socket-level protocol handling and HTTP message formatting MUST be validated through contract tests.
- **R-CONTRACT-005** MUST: Error handling contracts including exception types, error messages, and failure modes exposed through public interfaces MUST be validated through dedicated test functions.
- **R-CONTRACT-006** SHOULD: Test modules SHOULD be organized by functional domain matching the source module structure (e.g., `test_connectionpool.py` tests `src/urllib3/connectionpool.py`).
- **R-CONTRACT-007** SHOULD: Shared fixtures including server configuration, certificate generation (trustme.CA), and loopback host parameterization SHOULD be defined in `conftest.py`.
- **R-CONTRACT-008** SHOULD: Socket-level integration tests SHOULD use handler functions that simulate protocol-level interactions.
- **R-CONTRACT-009** SHOULD: Test function names SHOULD follow the convention `test_<contract_being_validated>` with descriptive names documenting the specific API behavior under test.
- **R-CONTRACT-010** MAY: Legacy test modules written before pytest adoption MAY use `unittest.TestCase`-based test classes (EXC-001).
- **R-CONTRACT-011** MAY: Platform-specific or environment-specific tests MAY skip contract validation using `pytest.skip` when required dependencies are unavailable (EXC-002).

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
- All test modules in `test/` directory contain test functions following the `test_*` naming convention and use pytest as the test framework
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary
- New public APIs include corresponding contract tests in the same commit
- Socket-level integration tests use pytest fixtures for proper resource cleanup and leverage `pytest.skip` for platform-specific exclusion

<enforcement>
Clause MUST NOT skip or defer verification. Contract tests MUST pass before merge. Code review MUST verify that new public APIs include corresponding contract tests. CI pipeline MUST execute pytest with mandatory test pass requirement.
</enforcement>
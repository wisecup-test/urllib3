# Standardize Public API Contract Testing with Pytest-Based Test Functions: Contract Tests Use

These rules are ALWAYS ACTIVE for all public API contract tests in the urllib3 codebase, particularly test modules validating public interfaces from core modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-CONTRACT-001** MAY: Contract tests MAY use parameterized fixtures (@pytest.fixture(params=...)) to validate behavior across multiple input variations or configuration scenarios.

### Verify

```bash
# Count test functions following test_* naming convention
grep -r "^def test_" test/ | wc -l

# Collect pytest test functions
pytest --collect-only -q test/ | grep "<Function" | wc -l

# Count pytest fixture definitions in conftest.py
grep -r "@pytest.fixture" test/conftest.py | wc -l

# Run contract tests for key API domains
pytest test/ -v --tb=short -k "test_proxy or test_connection or test_filepost"
```

**Accept when:**
- All test modules in test/ directory contain test functions following the test_* naming convention and use pytest as the test framework
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary
- Parameterized fixtures are used where appropriate to validate behavior across multiple input variations or configuration scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. All contract tests MUST execute successfully via pytest before accepting changes to public API surfaces. Code review MUST verify that new public APIs include corresponding contract tests. CI pipeline MUST enforce mandatory test pass requirement before merge.
</enforcement>
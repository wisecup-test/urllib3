# Standardize Public API Contract Testing with Pytest-Based Test Functions: Test Functions Use

These rules are ALWAYS ACTIVE for all test files in the `test/` directory that validate public API contracts for urllib3 modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions.

### Rules

- **R-TEST-001** SHOULD: Test functions SHOULD use descriptive names that clearly indicate the contract being validated (e.g., `test_match_hostname_no_cert`, `test_redirects_disabled_for_pool_manager_with_0`).

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
- Test function names are descriptive and document the specific API behavior or contract being validated

<enforcement>
Claude Code MUST NOT skip or defer verification. All test functions SHOULD follow the naming convention and clearly indicate the contract being validated. Violations must be flagged during code review and CI pipeline execution.
</enforcement>
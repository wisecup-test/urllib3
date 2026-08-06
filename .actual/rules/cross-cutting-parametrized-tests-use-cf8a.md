# Standardize pytest as Primary Test Framework for HTTP Client Testing: Parametrized Tests Use

These rules are ALWAYS ACTIVE for all Python test files in `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-PYTEST-001** SHOULD: Parametrized tests SHOULD use `@pytest.fixture(params=...)` or `@pytest.mark.parametrize` for testing multiple input combinations.

### Verify

```bash
# Count pytest imports in test files
grep -r "^import pytest" test/ | wc -l

# Count pytest fixture decorators
grep -r "@pytest.fixture" test/ | wc -l

# Count pytest.raises usage
grep -r "pytest.raises" test/ | wc -l

# Count test files following naming convention
find test/ -name 'test_*.py' -type f | wc -l

# Execute pytest discovery without running tests
pytest test/ --collect-only

# Run full test suite
pytest test/
```

**Accept when:**
- All test files in `test/` and `test/with_dummyserver/` directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via `pytest test/` successfully discovers and runs all test modules following pytest naming conventions
- conftest.py files contain shared fixtures using `@pytest.fixture` decorator and are properly discovered by pytest
- Parametrized tests use either `@pytest.fixture(params=...)` or `@pytest.mark.parametrize` decorators
- pytest --collect-only reports no collection errors or warnings

<enforcement>
Claude Code MUST NOT skip or defer verification of parametrized test patterns. All new test files and modifications to existing parametrized tests MUST conform to these rules before acceptance.
</enforcement>
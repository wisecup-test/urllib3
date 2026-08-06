# Standardize pytest as Primary Test Framework for HTTP Client Testing: Exception Testing Use

These rules are ALWAYS ACTIVE for all Python test files in the `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-EX-001** MUST: Exception testing MUST use `pytest.raises()` context manager rather than try-except blocks.

### Verify

```bash
# Count pytest imports in test files
grep -r "^import pytest" test/ | wc -l

# Count pytest.fixture decorators
grep -r "@pytest.fixture" test/ | wc -l

# Count pytest.raises usage for exception testing
grep -r "pytest.raises" test/ | wc -l

# Count test files following pytest naming convention
find test/ -name 'test_*.py' -type f | wc -l

# Verify test execution with pytest
pytest test/ --collect-only
```

**Accept when:**
- All test files in `test/` and `test/with_dummyserver/` directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via `pytest test/` successfully discovers and runs all test modules following pytest naming conventions
- conftest.py files contain shared fixtures using `@pytest.fixture` decorator and are properly discovered by pytest
- Exception assertions use `pytest.raises()` context manager exclusively (no try-except blocks for exception testing)

<enforcement>
Claude Code MUST NOT skip or defer verification of exception testing patterns. All exception assertions in test files MUST be reviewed to ensure compliance with R-EX-001.
</enforcement>
# Standardize pytest as Primary Test Framework for HTTP Client Testing: Test Fixtures Defined

These rules are ALWAYS ACTIVE for all Python test files in the `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixture/configuration files (conftest.py).

### Rules

- **R-PYTEST-FIXTURES-001** MUST: Test fixtures MUST be defined using `@pytest.fixture` decorator and shared fixtures MUST be placed in `conftest.py`.

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

# Verify pytest can discover and collect all tests
pytest test/ --collect-only
```

**Accept when:**
- All test files in `test/` and `test/with_dummyserver/` directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via `pytest test/` successfully discovers and runs all test modules following pytest naming conventions
- `conftest.py` files contain shared fixtures using `@pytest.fixture` decorator and are properly discovered by pytest
- No test fixtures are defined using unittest-style `setUp`/`tearDown` methods (except in legacy files with documented EXC-001 exception)
- All fixture definitions in `conftest.py` use the `@pytest.fixture` decorator

<enforcement>
Claude Code MUST NOT skip or defer verification of pytest fixture standardization. All new test fixtures MUST use `@pytest.fixture` decorator. All shared fixtures MUST be placed in `conftest.py` at the appropriate directory level.
</enforcement>
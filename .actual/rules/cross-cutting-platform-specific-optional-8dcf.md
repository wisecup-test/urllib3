# Standardize pytest as Primary Test Framework for HTTP Client Testing: Platform Specific Optional

These rules are ALWAYS ACTIVE for all Python test files in `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-PYTEST-001** SHOULD: Platform-specific or optional dependency tests SHOULD use `pytest.skip()` to conditionally skip tests when requirements are not met.
- **R-PYTEST-002** MUST: All test files in scope MUST use pytest as the test runner and follow pytest naming conventions (`test_*.py` for files, `test_*` for functions).
- **R-PYTEST-003** MUST: Shared test fixtures MUST be defined in `conftest.py` files using the `@pytest.fixture` decorator.
- **R-PYTEST-004** SHOULD: Test categorization SHOULD use pytest markers (`@pytest.mark.integration`, `@pytest.mark.requires_ssl`) to enable selective test execution.
- **R-PYTEST-005** MAY: Legacy test files inheriting from `unittest.TestCase` MAY retain unittest-style assertions while using pytest as the runner (EXC-001).

### Verify

```bash
# Count pytest imports across test suite
grep -r "^import pytest" test/ | wc -l

# Count pytest fixture definitions
grep -r "@pytest.fixture" test/ | wc -l

# Count pytest.raises usage
grep -r "pytest.raises" test/ | wc -l

# Count test files following naming convention
find test/ -name 'test_*.py' -type f | wc -l

# Verify pytest test discovery succeeds
pytest test/ --collect-only

# Verify conftest.py files are properly discovered
find test/ -name 'conftest.py' -type f
```

**Accept when:**
- All test files in `test/` and `test/with_dummyserver/` directories use pytest import and pytest-specific features (fixtures, raises, marks).
- Test execution via `pytest test/` successfully discovers and runs all test modules following pytest naming conventions.
- `conftest.py` files contain shared fixtures using `@pytest.fixture` decorator and are properly discovered by pytest.
- Platform-specific and optional dependency tests use `pytest.skip()` for conditional skipping.
- Test markers are applied to categorize tests for selective execution.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for test code within scope. Violations must be flagged during code review and CI pipeline execution.
</enforcement>
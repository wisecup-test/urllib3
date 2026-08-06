# Standardize pytest as Primary Test Framework for HTTP Client Testing: Test Organization Separate

These rules are ALWAYS ACTIVE for all Python test files in the `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-PYTEST-001** SHOULD: Test organization SHOULD separate unit tests (test/*.py) from integration tests (test/with_dummyserver/*.py).
- **R-PYTEST-002** MUST: All test files in scope MUST use pytest as the test runner and follow pytest naming conventions (test_*.py for files, test_* for functions).
- **R-PYTEST-003** MUST: Shared fixtures MUST be defined in conftest.py files using the @pytest.fixture decorator at appropriate directory levels.
- **R-PYTEST-004** SHOULD: Test categorization SHOULD use pytest markers (@pytest.mark.integration, @pytest.mark.requires_ssl) to enable selective test execution.
- **R-PYTEST-005** MUST: pytest configuration MUST be explicitly defined in pyproject.toml or pytest.ini with testpaths=['test'], python_files=['test_*.py'], python_functions=['test_*'].
- **R-PYTEST-006** MAY: Legacy test files that inherit from unittest.TestCase MAY retain unittest-style assertions while using pytest as the runner (EXC-001).

### Verify

```bash
# Count pytest imports in test files
grep -r "^import pytest" test/ | wc -l

# Count pytest fixture definitions
grep -r "@pytest.fixture" test/ | wc -l

# Count pytest.raises usage
grep -r "pytest.raises" test/ | wc -l

# Count test files following naming convention
find test/ -name 'test_*.py' -type f | wc -l

# Verify pytest can discover and collect all tests
pytest test/ --collect-only

# Verify pytest execution succeeds
pytest test/
```

**Accept when:**
- All test files in test/ and test/with_dummyserver/ directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via 'pytest test/' successfully discovers and runs all test modules following pytest naming conventions
- conftest.py files contain shared fixtures using @pytest.fixture decorator and are properly discovered by pytest
- pytest configuration is explicitly defined in pyproject.toml or pytest.ini with consistent discovery patterns
- Test markers are applied to categorize tests (integration, unit, requires_ssl, etc.)

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files MUST follow pytest conventions. CI pipeline MUST execute pytest and fail on test discovery errors or test failures. Code review MUST verify new test files follow pytest conventions and naming patterns.
</enforcement>
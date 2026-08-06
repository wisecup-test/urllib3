# Standardize pytest as Primary Test Framework for HTTP Client Testing: Tests Use Unittest

These rules are ALWAYS ACTIVE for all Python test files in `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-PYTEST-001** MUST: Use pytest as the primary test framework for all test files.
- **R-PYTEST-002** MAY: Tests MAY use unittest.mock for mocking when needed, as pytest is compatible with unittest-style assertions.
- **R-PYTEST-003** MUST: All test files follow pytest naming conventions (test_*.py for files, test_* for functions).
- **R-PYTEST-004** MUST: Shared fixtures MUST be defined in conftest.py using @pytest.fixture decorator.
- **R-PYTEST-005** SHOULD: Use pytest parametrization (@pytest.mark.parametrize) to reduce test code duplication when testing multiple input combinations.
- **R-PYTEST-006** SHOULD: Use pytest markers (@pytest.mark.integration, @pytest.mark.requires_ssl) to categorize tests and enable selective test execution.
- **R-PYTEST-007** MUST: Configure pytest in pyproject.toml or pytest.ini with explicit testpaths=['test'], python_files=['test_*.py'], python_functions=['test_*'].
- **R-PYTEST-008** SHOULD: Keep fixture scope minimal and use explicit fixture parameters rather than autouse fixtures.
- **R-PYTEST-EXC-001** MAY: Legacy test files that inherit from unittest.TestCase may retain unittest-style assertions while using pytest as the runner.

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

# Verify pytest execution
pytest test/ --collect-only

# Verify conftest.py discovery
find test/ -name 'conftest.py' -type f
```

**Accept when:**
- All test files in test/ and test/with_dummyserver/ directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via 'pytest test/' successfully discovers and runs all test modules following pytest naming conventions
- conftest.py files contain shared fixtures using @pytest.fixture decorator and are properly discovered by pytest
- pytest.ini or pyproject.toml contains explicit testpaths and discovery configuration
- Test files follow test_*.py naming convention for files and test_* for function names

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files MUST be validated against pytest conventions and naming patterns before acceptance.
</enforcement>
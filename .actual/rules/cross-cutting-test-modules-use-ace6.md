# Standardize pytest as Primary Test Framework for HTTP Client Testing: Test Modules Use

These rules are ALWAYS ACTIVE for all Python test files in the `test/` and `test/with_dummyserver/` directories, test automation scripts (noxfile.py), continuous integration configurations, and test fixtures and configuration files (conftest.py).

### Rules

- **R-PYTEST-001** MUST: All test modules MUST use pytest as the primary test framework and follow pytest discovery conventions (test_*.py files, test_* function names).
- **R-PYTEST-002** MUST: Test files in test/ and test/with_dummyserver/ directories MUST use pytest import and pytest-specific features (fixtures, raises, marks).
- **R-PYTEST-003** MUST: Shared fixtures MUST be placed in conftest.py at appropriate directory levels using @pytest.fixture decorator.
- **R-PYTEST-004** SHOULD: Use pytest markers (@pytest.mark.integration, @pytest.mark.requires_ssl) to categorize tests and enable selective test execution.
- **R-PYTEST-005** SHOULD: Configure pytest in pyproject.toml or pytest.ini with explicit testpaths=['test'], python_files=['test_*.py'], python_functions=['test_*'] to ensure consistent discovery.
- **R-PYTEST-EXC-001** MAY: Legacy test files that inherit from unittest.TestCase may retain unittest-style assertions while using pytest as the runner.

### Verify

```bash
# Count pytest imports in test files
grep -r "^import pytest" test/ | wc -l

# Count pytest fixtures
grep -r "@pytest.fixture" test/ | wc -l

# Count pytest.raises usage
grep -r "pytest.raises" test/ | wc -l

# Count test_*.py files
find test/ -name 'test_*.py' -type f | wc -l

# Verify pytest can discover and collect all tests
pytest test/ --collect-only

# Execute full test suite
pytest test/
```

**Accept when:**
- All test files in test/ and test/with_dummyserver/ directories use pytest import and pytest-specific features (fixtures, raises, marks).
- Test execution via 'pytest test/' successfully discovers and runs all test modules following pytest naming conventions.
- conftest.py files contain shared fixtures using @pytest.fixture decorator and are properly discovered by pytest.
- pytest --collect-only completes without errors and identifies all test modules.
- All tests pass when executed with pytest.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for test modules in scope. Violations must be caught during code review and CI pipeline execution.
</enforcement>
# Standardize pytest as Primary Testing Framework with urllib3 Core Library Integration: Integration Test Sessions

These rules are ALWAYS ACTIVE for all test files in the `test/` directory and subdirectories, including test configuration files (conftest.py, noxfile.py), unit tests for urllib3 core modules, integration tests with dummy servers, and socket-level protocol tests.

### Rules

- **R-PYTEST-001** MUST: All test files in `test/` directory and subdirectories SHALL import pytest explicitly and use pytest-style assertions or fixtures rather than unittest patterns.
- **R-PYTEST-002** MUST: Test files validating urllib3 functionality SHALL import at least one urllib3 core library (urllib3.exceptions, urllib3.poolmanager, urllib3.fields, urllib3.connection) directly in the test module.
- **R-PYTEST-003** SHOULD: Integration test sessions SHOULD be orchestrated through noxfile.py with dedicated test environments (test_integration, test_min_pyopenssl, test_brotlipy) with explicit dependency specifications.
- **R-PYTEST-004** MUST: Reusable test fixtures SHALL be defined in conftest.py with @pytest.fixture decorator and appropriate scope (function, module, session) with clear parameter types and explicit documentation of fixture dependencies.
- **R-PYTEST-005** MUST: Exception testing SHALL use pytest.raises() for exception validation rather than unittest assertRaises patterns.
- **R-PYTEST-006** SHOULD: Test functions SHOULD be named descriptively following the pattern test_<behavior>_<condition> (e.g., test_proxy_headers, test_match_hostname_no_cert) to document API contracts.
- **R-PYTEST-007** SHOULD: Socket-level tests SHOULD use sock.send() with properly formatted HTTP responses and validate protocol behavior at the byte level with clear separation between protocol versions (HTTP/1.1 vs HTTP/2).
- **R-PYTEST-008** MAY: Legacy test modules MAY require unittest.TestCase for compatibility with existing test infrastructure or CI pipelines only when documented exception EXC-001 is approved.
- **R-PYTEST-009** MAY: Platform-specific tests MAY require alternative testing approaches due to OS limitations (e.g., Windows-specific socket behavior) only when documented exception EXC-002 is approved.

### Verify

```bash
# Count pytest imports across test directory
grep -r "^import pytest" test/ | wc -l

# Count urllib3 core library imports in tests
grep -r "urllib3\.exceptions\|urllib3\.poolmanager\|urllib3\.fields\|urllib3\.connection" test/ | wc -l

# Count pytest fixtures defined in conftest.py
grep -r "@pytest\.fixture" test/conftest.py | wc -l

# Collect all test functions via pytest
python -m pytest test/ --collect-only | grep "<Function" | wc -l

# Verify pytest can collect tests without import errors
python -m pytest test/ --collect-only -q
```

**Accept when:**
- All test modules in `test/` directory import pytest and use pytest-style assertions or fixtures
- Test files testing urllib3 functionality import at least one urllib3 core library (exceptions, poolmanager, fields, connection)
- conftest.py contains pytest fixtures for server configurations with @pytest.fixture decorator
- pytest --collect-only successfully discovers and collects all test functions without import errors
- noxfile.py defines dedicated test sessions (test_integration, test_min_pyopenssl, test_brotlipy) with explicit dependency specifications

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST run pytest with coverage reporting and fail on import errors or test collection failures. Code review MUST verify new test files import pytest and urllib3 core libraries appropriately. Pre-commit hooks MUST validate test file structure and pytest import presence. Violations result in CI build failure and code review blocking merge unless documented exception is approved.
</enforcement>
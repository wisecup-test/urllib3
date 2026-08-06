# Standardize pytest as Primary Test Framework for HTTP Client Testing: Parametrized Tests Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 20 test files with 92.19% confidence showing consistent use of pytest as the test framework across multiple test domains including proxy management, file posting, socket-level operations, connection pooling, and HTTP/2 connections
- Test files demonstrate pytest-specific patterns including fixtures (@pytest.fixture), parametrization, exception assertions (pytest.raises), and test discovery conventions (test_* function naming)
- The test suite covers both unit tests (test_fields.py, test_filepost.py) and integration tests with dummy servers (test_socketlevel.py, test_poolmanager.py, test_proxy_poolmanager.py), requiring a framework that supports both paradigms
- Evidence shows pytest integration with other testing utilities including unittest.mock, contextlib for resource management, and trustme for certificate generation in conftest.py
- The noxfile.py demonstrates pytest invocation patterns for test automation including test_integration, test_min_pyopenssl, and test_brotlipy sessions

## Problem Statement

The project requires a consistent, feature-rich test framework capable of handling diverse testing scenarios including unit tests, integration tests with network servers, SSL/TLS certificate validation, proxy configurations, and HTTP protocol compliance across multiple Python versions and optional dependencies.

## Decision

1. SHOULD: Parametrized tests SHOULD use @pytest.fixture(params=...) or @pytest.mark.parametrize for testing multiple input combinations

## Policy Block

- SHOULD Parametrized tests SHOULD use @pytest.fixture(params=...) or @pytest.mark.parametrize for testing multiple input combinations

In scope:
- All Python test files in test/ and test/with_dummyserver/ directories
- Test automation scripts (noxfile.py) that invoke test runners
- Continuous integration configurations that execute test suites
- Test fixtures and configuration files (conftest.py)

Out of scope:
- Documentation examples that demonstrate library usage (may use simple assertions)
- Performance benchmarking scripts that may use alternative measurement frameworks
- Development utility scripts not part of the test suite

Exceptions:
- EXC-001: Legacy test files that inherit from unittest.TestCase may retain unittest-style assertions while using pytest as the runner

## Rationale

- Evidence from 20 test files shows pytest is already the de facto standard with 92.19% confidence, indicating strong existing adoption and team familiarity
- pytest provides essential features demonstrated in the evidence: parametrization for testing multiple scenarios (loopback_host fixture with localhost/127.0.0.1/::1), powerful fixture system for managing test servers and SSL contexts, and flexible test discovery
- The conftest.py file demonstrates pytest's fixture sharing capabilities with san_server, no_san_server, and loopback_host fixtures that manage complex test infrastructure including trustme certificate generation and server lifecycle
- pytest's compatibility with unittest-style tests (seen in test_queue_monkeypatch.py and test_connectionpool.py) enables gradual migration and coexistence of different test styles

## Consequences

Positive:
- Consistent test execution and reporting across all test modules with unified pytest output format
- Powerful fixture system enables clean separation of test setup/teardown logic and promotes reusable test infrastructure (server fixtures, SSL context fixtures)
- Parametrization support reduces test code duplication when testing multiple input combinations or configurations
- Rich plugin ecosystem and active community support for extending test capabilities (pytest-cov, pytest-timeout, pytest-xdist for parallel execution)

Negative:
- Additional dependency on pytest package and its dependencies increases installation footprint
- Team members unfamiliar with pytest fixtures and advanced features face learning curve compared to simple unittest assertions
- pytest's implicit test discovery and fixture injection can make test execution flow less explicit than traditional unittest setUp/tearDown methods
- Debugging test failures may require understanding pytest's fixture resolution and parametrization mechanics

## Alternatives

- Use standard library unittest exclusively without pytest (rejected)
  Rejected because: unittest lacks parametrization support, has more verbose fixture management (setUp/tearDown methods), and provides less flexible test discovery. Evidence shows pytest is already deeply integrated with 20 files using pytest-specific features.
  When valid: For projects with no external dependencies requirement or very simple test suites
- Use nose2 as test framework (rejected)
  Rejected because: nose2 has limited active development and smaller community compared to pytest. No evidence of nose2 usage in the codebase, requiring complete test suite migration.
  When valid: For projects already invested in nose/nose2 with extensive custom plugins
- Mix multiple test frameworks (pytest, unittest, nose) based on test type (rejected)
  Rejected because: Multiple frameworks increase complexity, require maintaining multiple test execution paths, and create inconsistent developer experience. Evidence shows pytest can handle all test types (unit, integration, socket-level).
  When valid: Never recommended; standardization provides better maintainability

## Risks

- pytest version incompatibilities across Python versions (2.7, 3.x) may cause test failures or require version pinning
  Mitigation: Pin pytest version ranges in test requirements, use nox sessions to test against multiple pytest versions, monitor pytest changelog for breaking changes
  Owner: Test infrastructure team
- Complex fixture dependencies in conftest.py may create implicit coupling and make test failures harder to debug
  Mitigation: Document fixture dependencies clearly, keep fixture scope minimal, use explicit fixture parameters rather than autouse fixtures, provide fixture documentation in conftest.py
  Owner: Test maintainers
- pytest's implicit test discovery may execute unintended files or skip tests due to naming convention violations
  Mitigation: Configure pytest discovery patterns explicitly in pytest.ini or pyproject.toml, enforce test_*.py naming in code review, use pytest --collect-only to verify test discovery
  Owner: Engineering team

## Implementation Notes

- Configure pytest in pyproject.toml or pytest.ini with explicit testpaths=['test'], python_files=['test_*.py'], python_functions=['test_*'] to ensure consistent discovery
- Place shared fixtures in conftest.py at appropriate directory levels (test/conftest.py for all tests, test/with_dummyserver/conftest.py for integration-specific fixtures)
- Use pytest markers (@pytest.mark.integration, @pytest.mark.requires_ssl) to categorize tests and enable selective test execution
- Leverage pytest's --collect-only flag during development to verify test discovery without execution, and use -v flag for detailed test output during debugging

## Continuation Context


Verify commands:
- grep -r "^import pytest" test/ | wc -l
- grep -r "@pytest.fixture" test/ | wc -l
- grep -r "pytest.raises" test/ | wc -l
- find test/ -name 'test_*.py' -type f | wc -l

Accept when:
- All test files in test/ and test/with_dummyserver/ directories use pytest import and pytest-specific features (fixtures, raises, marks)
- Test execution via 'pytest test/' successfully discovers and runs all test modules following pytest naming conventions
- conftest.py files contain shared fixtures using @pytest.fixture decorator and are properly discovered by pytest

## Enforcement

- Verified by: Automated CI pipeline executes pytest and fails on test discovery errors or test failures
- Verified by: Code review checklist verifies new test files follow pytest conventions and naming patterns
- Verified by: Pre-commit hooks or linting tools check for pytest import and fixture usage in test files
- Violation handling: CI pipeline fails if tests do not follow pytest discovery conventions or if pytest execution fails
- Violation handling: Code review blocks merge if new tests use alternative frameworks without documented exception approval
- Violation handling: Test coverage reports flag untested code to ensure pytest-based tests provide adequate coverage
- Exception process: Request exception approval from test maintainer with justification for alternative approach
- Exception process: Document exception in test module docstring explaining why pytest conventions are not followed
- Exception process: Review exceptions quarterly to determine if they can be migrated to standard pytest patterns
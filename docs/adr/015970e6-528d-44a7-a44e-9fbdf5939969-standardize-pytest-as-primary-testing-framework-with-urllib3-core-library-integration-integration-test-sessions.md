# Standardize pytest as Primary Testing Framework with urllib3 Core Library Integration: Integration Test Sessions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase demonstrates consistent use of pytest as the testing framework across 27 test files, including test_proxymanager.py, test_filepost.py, test_connection.py, and test_http2_connection.py
- Test files systematically import and utilize urllib3 core libraries (urllib3.exceptions, urllib3.poolmanager, urllib3.fields, urllib3.connection) alongside pytest fixtures and assertions
- The testing infrastructure includes specialized test configurations in conftest.py with pytest fixtures for loopback hosts, SAN servers, and no-SAN servers using trustme CA certificates
- Integration testing patterns are evident in noxfile.py with dedicated test sessions (test_integration, test_min_pyopenssl, test_brotlipy) orchestrated through nox
- Socket-level and low-level HTTP testing patterns in test_socketlevel.py demonstrate direct protocol testing with manual HTTP response construction and message queue boundaries

## Problem Statement

The project requires a consistent, maintainable testing strategy that can validate HTTP client behavior across multiple layers (unit, integration, socket-level) while ensuring compatibility with urllib3's core libraries and exception handling patterns. Without standardized testing framework adoption, test maintenance becomes fragmented and verification of complex networking behaviors becomes inconsistent.

## Decision

1. SHOULD: Integration test sessions SHOULD be orchestrated through noxfile.py with dedicated test environments (test_integration, test_min_pyopenssl, test_brotlipy)

## Policy Block

- SHOULD Integration test sessions SHOULD be orchestrated through noxfile.py with dedicated test environments (test_integration, test_min_pyopenssl, test_brotlipy)

In scope:
- All test files in test/ directory and subdirectories
- Test configuration files (conftest.py, noxfile.py)
- Unit tests for urllib3 core modules (connectionpool, fields, filepost, proxymanager)
- Integration tests with dummy servers (test/with_dummyserver/)
- Socket-level and protocol-level tests
- Exception and error handling validation tests

Out of scope:
- Production source code in src/urllib3/
- Documentation and example code
- Build and packaging scripts (except noxfile.py)
- Third-party test utilities and fixtures from external packages

Exceptions:
- EXC-001: Legacy test modules require unittest.TestCase for compatibility with existing test infrastructure or CI pipelines
- EXC-002: Platform-specific tests require alternative testing approaches due to OS limitations (e.g., Windows-specific socket behavior)

## Rationale

- Evidence from 27 test files shows consistent pytest adoption with 91.95% confidence, indicating an established pattern rather than isolated usage
- The integration of pytest with urllib3 core libraries enables comprehensive testing of HTTP client behavior, exception handling, and connection pooling across multiple abstraction layers
- Centralized fixture management in conftest.py with trustme CA integration demonstrates a mature testing infrastructure that supports TLS/SSL testing scenarios
- The noxfile.py orchestration pattern enables reproducible test environments for different dependency configurations (PyOpenSSL, Brotli), ensuring compatibility testing

## Consequences

Positive:
- Consistent testing framework across all test modules reduces cognitive load and improves test maintainability
- pytest fixtures enable reusable test infrastructure (server configs, certificates, loopback hosts) reducing test code duplication
- Integration with urllib3 core libraries ensures tests validate actual production code paths and exception handling
- Nox-based test orchestration provides reproducible test environments for different dependency configurations and Python versions

Negative:
- Requires pytest as a mandatory test dependency, increasing the dependency footprint for development environments
- Mixed unittest.TestCase and pytest-style tests create inconsistency in some legacy test modules
- Socket-level testing with manual HTTP response construction increases test complexity and maintenance burden
- Platform-specific test fixtures (IPv6 loopback, SSL contexts) may require conditional skipping on some environments

## Alternatives

- Use unittest exclusively as the standard library testing framework without pytest (rejected)
  Rejected because: unittest lacks advanced fixture management, parameterization, and plugin ecosystem that pytest provides. Evidence shows pytest is already deeply integrated with 27 test files using pytest-specific features.
  When valid: Only valid for minimal projects with no complex fixture requirements or when avoiding external dependencies is critical
- Adopt nose2 or another unittest-compatible test runner (rejected)
  Rejected because: nose2 has limited active development and smaller ecosystem compared to pytest. Migration would require rewriting existing pytest fixtures and parameterized tests.
  When valid: Valid only if maintaining strict unittest compatibility is required for organizational policy
- Use multiple testing frameworks based on test type (unittest for unit tests, pytest for integration) (rejected)
  Rejected because: Multiple frameworks increase complexity, fragment test infrastructure, and create inconsistent test patterns. Evidence shows pytest handles all test types effectively.
  When valid: Valid only during migration period when gradually transitioning from unittest to pytest

## Risks

- pytest version incompatibilities or breaking changes in future releases could impact test execution across 27+ test files
  Mitigation: Pin pytest version in requirements-dev.txt and test against multiple pytest versions in CI. Monitor pytest release notes and test with release candidates.
  Owner: Test infrastructure team
- Complex fixture dependencies in conftest.py (trustme CA, server configs) may create hidden test interdependencies and flaky tests
  Mitigation: Document fixture dependencies explicitly, use fixture scope appropriately (function/module/session), and implement fixture cleanup with proper teardown
  Owner: Engineering team
- Socket-level tests with manual HTTP response construction may become brittle when HTTP protocol details change or when testing HTTP/2 vs HTTP/1.1
  Mitigation: Encapsulate protocol-specific response construction in helper functions, maintain clear separation between protocol versions, and document protocol assumptions
  Owner: Protocol testing team

## Implementation Notes

- Import pytest explicitly in all test modules and use pytest.raises() for exception testing rather than unittest assertRaises
- Define reusable fixtures in conftest.py with appropriate scope (function, module, session) and use @pytest.fixture decorator with clear parameter types
- Import urllib3 core libraries (urllib3.exceptions, urllib3.poolmanager, urllib3.fields, urllib3.connection) directly in test modules that validate those components
- Use noxfile.py to define test sessions for different environments (test_integration, test_min_pyopenssl, test_brotlipy) with explicit dependency specifications
- For socket-level tests, use sock.send() with properly formatted HTTP responses and validate protocol behavior at the byte level
- Name test functions descriptively following the pattern test_<behavior>_<condition> (e.g., test_proxy_headers, test_match_hostname_no_cert) to document API contracts

## Continuation Context


Verify commands:
- grep -r "^import pytest" test/ | wc -l
- grep -r "urllib3\.exceptions\|urllib3\.poolmanager\|urllib3\.fields\|urllib3\.connection" test/ | wc -l
- grep -r "@pytest\.fixture" test/conftest.py | wc -l
- python -m pytest test/ --collect-only | grep "<Function" | wc -l

Accept when:
- All test modules in test/ directory import pytest and use pytest-style assertions or fixtures
- Test files testing urllib3 functionality import at least one urllib3 core library (exceptions, poolmanager, fields, connection)
- conftest.py contains pytest fixtures for server configurations with @pytest.fixture decorator
- pytest --collect-only successfully discovers and collects all test functions without import errors

## Enforcement

- Verified by: CI pipeline runs pytest with coverage reporting and fails on import errors or test collection failures
- Verified by: Code review checklist verifies new test files import pytest and urllib3 core libraries appropriately
- Verified by: Pre-commit hooks validate test file structure and pytest import presence
- Verified by: Automated grep-based verification in CI checks for pytest and urllib3 import patterns
- Violation handling: CI build fails if test files do not import pytest or if pytest collection fails
- Violation handling: Code review blocks merge if new tests use unittest without documented exception approval
- Violation handling: Automated linting flags test files missing pytest imports or using deprecated unittest patterns
- Violation handling: Test coverage reports highlight untested urllib3 core library integration points
- Exception process: Submit exception request to test lead with documented rationale for unittest usage or alternative testing approach
- Exception process: Document platform-specific constraints or legacy compatibility requirements in test module docstring
- Exception process: Create GitHub issue tracking migration plan from unittest to pytest for approved exceptions
- Exception process: Review exceptions quarterly to assess migration progress and update exception status
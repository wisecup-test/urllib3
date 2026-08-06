# Standardize pytest Fixture-Based Test Configuration with Parameterized Hosts: Connection State Tracking

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- Test infrastructure requires consistent access patterns for network-bound resources across IPv4 and IPv6 loopback addresses
- pytest fixtures provide parameterized test execution with loopback_host fixture yielding localhost, 127.0.0.1, and ::1 variants
- TLS certificate generation and server configuration must coordinate with parameterized host values through trustme CA and server_cert fixtures
- Test collection and execution hooks (pytest_addoption, pytest_collection_modifyitems) establish public contracts for test configuration
- Connection pooling patterns aggregate connections via connections.add(conn) and track TLS versions through tls_versions.add(_sock.version())

## Problem Statement

Test suites accessing network resources need a standardized mechanism to validate behavior across multiple loopback addresses (IPv4, IPv6) while coordinating certificate generation, server lifecycle, and connection state tracking without duplicating test logic or hardcoding host values.

## Decision

1. MUST: Connection state tracking MUST use set-based aggregation patterns (connections.add(conn), tls_versions.add(_sock.version()))

## Policy Block

- MUST Connection state tracking MUST use set-based aggregation patterns (connections.add(conn), tls_versions.add(_sock.version()))

In scope:
- pytest-based test suites with network resource access
- TLS/SSL certificate generation and validation tests
- Connection pooling and socket-level test scenarios
- Fixtures in conftest.py establishing test infrastructure

Out of scope:
- Production application code outside test directories
- Non-pytest test frameworks (unittest, nose)
- Integration tests using external network addresses
- Mock-based tests without actual network socket creation

## Rationale

- Evidence shows 2 files (test/conftest.py, test/test_poolmanager.py) implementing parameterized fixture patterns with 91.85% confidence, establishing a consistent access pattern
- Parameterized loopback_host fixture eliminates test duplication by executing identical test logic across IPv4 and IPv6 variants automatically
- Coordination between trustme CA, server_cert fixtures, and parameterized hosts ensures certificate SAN/CN values match test execution context
- Set-based aggregation (connections.add, tls_versions.add) provides efficient state tracking across parameterized test executions

## Consequences

Positive:
- Test coverage automatically spans IPv4 and IPv6 loopback addresses without duplicating test functions
- Certificate generation coordinates with runtime host values, preventing SAN/CN mismatch errors
- Fixture-based dependency injection isolates test infrastructure from test logic, improving maintainability
- Public contracts (pytest_addoption, pytest_collection_modifyitems) enable consistent test configuration across modules

Negative:
- Parameterized fixtures increase test execution time by multiplying test runs across parameter combinations
- Fixture dependency chains (loopback_host → san_server → tests) create implicit coupling that may obscure test failures
- IPv6 skip logic adds conditional complexity to test execution paths
- Set-based state aggregation requires careful cleanup to prevent cross-test contamination

## Alternatives

- Hardcode single loopback address (127.0.0.1) in all tests (rejected)
  Rejected because: Eliminates IPv6 test coverage and prevents detection of address-family-specific bugs in network code
  When valid: When IPv6 support is explicitly out of scope for the application
- Duplicate test functions for each loopback address variant (rejected)
  Rejected because: Creates maintenance burden with 3x code duplication and increases risk of test logic divergence
  When valid: When tests require significantly different logic per address family
- Use unittest.TestCase with setUp/tearDown for server lifecycle (rejected)
  Rejected because: Loses pytest fixture parameterization capabilities and requires manual resource management
  When valid: When migrating existing unittest-based test suites with minimal refactoring

## Risks

- Parameterized fixtures may obscure which specific host value caused test failure in CI logs
  Mitigation: Configure pytest to display parameter values in test names using --verbose flag and ensure CI captures full pytest output
  Owner: engineering team
- IPv6 availability detection (HAS_IPV6) may produce false negatives on systems with disabled IPv6 stack
  Mitigation: Document IPv6 requirements in test README and provide explicit skip messages with pytest.skip() rationale
  Owner: engineering team
- Fixture-scoped resources (CA, certificates, servers) may leak if context managers fail to clean up properly
  Mitigation: Implement explicit cleanup in fixture finalizers and use pytest-timeout to detect hanging server threads
  Owner: engineering team

## Implementation Notes

- Define loopback_host fixture in conftest.py with @pytest.fixture(params=['localhost', '127.0.0.1', '::1']) to enable automatic parameterization
- Use typing.Generator[str] return type for fixtures yielding values and typing.Generator[ServerConfig] for complex configuration objects
- Coordinate certificate generation by passing loopback_host to trustme.CA.issue_cert() for SAN or issue_cert(common_name=loopback_host) for CN-only
- Wrap server lifecycle in context managers (run_server_in_thread) that yield ServerConfig and ensure cleanup on fixture teardown
- Implement HAS_IPV6 detection early in conftest.py and use pytest.skip() with descriptive messages for IPv6-dependent tests
- Use tmp_path_factory.mktemp() for certificate storage to ensure test isolation and automatic cleanup

## Continuation Context


Verify commands:
- grep -r '@pytest.fixture(params=' test/ | grep -E '(localhost|127\.0\.0\.1|::1)'
- grep -r 'trustme\.CA' test/ | grep 'issue_cert'
- grep -r '\.add\(' test/ | grep -E '(connections|tls_versions)'
- pytest --collect-only test/ | grep -E '\[(localhost|127\.0\.0\.1|::1)\]'

Accept when:
- Parameterized loopback_host fixture exists in conftest.py with all three address variants
- Certificate generation fixtures coordinate with loopback_host parameter values
- Test collection shows parameterized test instances for each loopback address variant
- Connection state tracking uses set-based aggregation patterns

## Enforcement

- Verified by: pytest --collect-only output inspection during CI to verify parameterized test generation
- Verified by: Code review verification that new network tests use loopback_host fixture parameter
- Verified by: grep-based static analysis checking for hardcoded loopback addresses in test files
- Violation handling: CI fails if new tests hardcode loopback addresses instead of using parameterized fixtures
- Violation handling: Code review blocks PRs that bypass fixture-based server lifecycle management
- Violation handling: Linting rules flag direct socket creation without fixture coordination
- Exception process: Document exception rationale in test docstring explaining why parameterization is inappropriate
- Exception process: Obtain approval from test infrastructure maintainer for non-parameterized network tests
- Exception process: Add # noqa comment with issue tracker reference for approved exceptions
# Adopt pytest Fixtures for Test Infrastructure Setup in Data Access Layer: Test Infrastructure Data

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test infrastructure requires parameterized setup of network resources (loopback hosts, TLS servers) to validate data access patterns across multiple protocol variants
- Connection pooling and manager lifecycle testing necessitates controlled fixture-based setup and teardown to ensure proper resource cleanup
- Test isolation demands that each test case receives fresh server configurations and certificate authorities without cross-contamination
- The codebase uses pytest as the test framework with fixtures for loopback_host, san_server, no_san_server, and contextmanager-based server lifecycle management

## Problem Statement

Testing data access patterns requires complex setup of network infrastructure, TLS certificates, and connection pools with proper lifecycle management. Without a standardized fixture-based approach, tests would duplicate setup logic, risk resource leaks, and fail to validate behavior across protocol variants (IPv4, IPv6, different TLS configurations).

## Decision

1. MUST: Test infrastructure for data access patterns MUST use pytest fixtures to manage lifecycle of network resources, servers, and connection pools

## Policy Block

- MUST Test infrastructure for data access patterns MUST use pytest fixtures to manage lifecycle of network resources, servers, and connection pools

In scope:
- All test modules validating HTTP/HTTPS connection pooling behavior
- Test cases requiring TLS certificate validation (SAN, no-SAN scenarios)
- Integration tests for PoolManager and connection lifecycle
- Tests exercising data access patterns with network I/O

Out of scope:
- Unit tests that mock network layers without actual socket connections
- Performance benchmarks requiring static server configurations
- Tests of pure data transformation logic without I/O

## Rationale

- Evidence shows pytest fixtures (loopback_host, san_server, no_san_server) are used in test/conftest.py with parameterization and contextmanager-based lifecycle management
- The pattern enables testing connection pooling behavior (connections.add(conn)) and TLS version collection (tls_versions.add(_sock.version())) across multiple network configurations
- Fixture-based setup reduces test code duplication and ensures consistent resource cleanup, as evidenced by the contextmanager pattern in run_server_in_thread
- The 91.85% confidence from 2 files (test/conftest.py, test/test_poolmanager.py) indicates this is an established pattern for data access layer testing

## Consequences

Positive:
- Test isolation improves through fixture-scoped resource management, preventing cross-test contamination
- Parameterized fixtures enable comprehensive validation across IPv4, IPv6, and various TLS configurations without code duplication
- Contextmanager-based server lifecycle ensures proper cleanup even when tests fail or raise exceptions
- Dynamic certificate generation via trustme eliminates security risks from hardcoded test credentials

Negative:
- Fixture setup overhead increases test execution time, particularly for tests requiring full server initialization
- Complex fixture dependency chains (loopback_host → san_server → test) can make test failures harder to debug
- Parameterized fixtures multiply test execution count, potentially causing CI pipeline slowdowns
- Developers must understand pytest fixture scoping and lifecycle to avoid resource leaks or unexpected sharing

## Alternatives

- Use unittest setUp/tearDown methods for test infrastructure initialization (rejected)
  Rejected because: setUp/tearDown lacks parameterization support and requires manual resource cleanup logic in each test class, increasing duplication and error risk
  When valid: When migrating legacy unittest-based test suites where fixture refactoring cost exceeds benefit
- Use module-level or session-level server instances shared across all tests (rejected)
  Rejected because: Shared server state prevents test isolation and can cause flaky tests when connection pools or TLS state is mutated
  When valid: For read-only integration tests against stable external services where setup cost is prohibitive
- Mock all network I/O to avoid fixture complexity (deferred)
  Rejected because: Mocking eliminates validation of actual connection pooling, TLS handshake, and socket behavior critical to data access patterns
  When valid: For unit tests of business logic that should not depend on network infrastructure

## Risks

- Fixture setup failures can cascade across test suite, causing widespread test failures unrelated to code changes
  Mitigation: Implement fixture health checks and clear error messages; use pytest-xdist for test isolation; add fixture-specific logging
  Owner: Test Infrastructure Team
- Parameterized fixtures with IPv6 may fail on CI environments without IPv6 support, causing false negatives
  Mitigation: Use pytest.skip() with HAS_IPV6 detection as shown in evidence; document IPv6 requirements in CI configuration
  Owner: DevOps Team
- Temporary certificate directories created by tmp_path_factory may accumulate if cleanup fails, consuming disk space
  Mitigation: Rely on pytest's automatic tmpdir cleanup; add monitoring for /tmp usage in CI; implement periodic cleanup jobs
  Owner: Engineering Team

## Implementation Notes

- Place shared fixtures in test/conftest.py to make them available across all test modules without explicit imports
- Use @pytest.fixture(params=[...]) for protocol variants and typing.Generator return types for contextmanager-based fixtures
- Implement server fixtures using 'with run_server_in_thread(...)' pattern to ensure cleanup via context manager __exit__
- Add HAS_IPV6 detection and pytest.skip() guards for IPv6-dependent tests to prevent failures on limited CI environments
- Use tmp_path_factory.mktemp() for certificate storage to leverage pytest's automatic temporary directory cleanup

## Continuation Context


Verify commands:
- grep -r '@pytest.fixture' test/ | grep -E '(loopback_host|san_server|no_san_server)' | wc -l
- grep -r 'run_server_in_thread' test/ | grep 'with.*as' | wc -l
- pytest test/test_poolmanager.py -v --collect-only | grep -E '(localhost|127.0.0.1|::1)' | wc -l

Accept when:
- At least 3 pytest fixtures are defined for network infrastructure setup (loopback_host, san_server, no_san_server)
- Server lifecycle fixtures use contextmanager pattern (with statement) for resource cleanup
- Parameterized fixtures generate multiple test variants for protocol coverage (IPv4, IPv6)

## Enforcement

- Verified by: Automated pytest collection analysis in CI pipeline to verify fixture usage
- Verified by: Code review checklist requiring fixture-based setup for new data access tests
- Verified by: Static analysis scanning for setUp/tearDown patterns in test files as anti-pattern indicator
- Violation handling: CI pipeline fails if new test files in test/ directory use unittest setUp/tearDown instead of pytest fixtures
- Violation handling: Code review blocks merge if data access tests lack proper fixture-based resource cleanup
- Violation handling: Automated linting warns on direct socket/server instantiation in test functions without fixture injection
- Exception process: Request exception via pull request comment with justification for alternative test approach
- Exception process: Test Infrastructure Team reviews exception request within 2 business days
- Exception process: Approved exceptions documented in test module docstring with expiration date and migration plan
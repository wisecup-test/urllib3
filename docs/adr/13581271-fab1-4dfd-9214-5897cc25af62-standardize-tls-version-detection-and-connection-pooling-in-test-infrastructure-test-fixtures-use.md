# Standardize TLS Version Detection and Connection Pooling in Test Infrastructure: Test Fixtures Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test infrastructure requires parameterized loopback host testing across IPv4 and IPv6 addresses (localhost, 127.0.0.1, ::1) to validate network protocol compatibility
- TLS certificate generation and validation testing necessitates dynamic server fixtures with both SAN and non-SAN certificates using trustme CA infrastructure
- Connection pooling behavior must be verified through test suites that track connection object lifecycle and reuse patterns across same-URL and many-URL scenarios
- The test framework uses pytest fixtures to coordinate server lifecycle, certificate provisioning, and connection pool state management for integration testing

## Problem Statement

Test infrastructure must validate TLS version negotiation, certificate validation patterns, and connection pool behavior across multiple network protocols and certificate configurations without requiring manual server setup or certificate management, while ensuring test isolation and reproducibility.

## Decision

1. SHOULD: Test fixtures SHOULD use pytest.TempPathFactory for certificate storage to ensure test isolation and cleanup

## Policy Block

- SHOULD Test fixtures SHOULD use pytest.TempPathFactory for certificate storage to ensure test isolation and cleanup

In scope:
- pytest-based test suites requiring TLS server fixtures
- Connection pool manager test scenarios (test_same_url, test_many_urls, test_manager_clear)
- Network protocol compatibility validation across IPv4 and IPv6
- Certificate validation testing with SAN and non-SAN configurations

Out of scope:
- Production server certificate management
- Non-pytest test frameworks
- Manual certificate provisioning workflows
- Connection pooling outside of test scenarios

## Rationale

- Evidence shows parameterized fixtures (@pytest.fixture(params=['localhost', '127.0.0.1', '::1'])) systematically testing network protocol variants, ensuring comprehensive coverage
- TLS version detection pattern (tls_versions.add(_sock.version())) and connection tracking (connections.add(conn)) demonstrate observable data access patterns for runtime state inspection
- trustme CA integration with temporary path factories provides reproducible certificate generation without external dependencies or manual key management
- Context manager patterns (run_server_in_thread) coordinate server lifecycle with test execution, preventing resource leaks and ensuring test isolation

## Consequences

Positive:
- Automated certificate generation eliminates manual certificate management overhead and reduces test setup complexity
- Parameterized loopback testing provides systematic coverage of IPv4 and IPv6 protocol variants with minimal test code duplication
- Observable connection pool state through data access patterns enables verification of connection reuse and lifecycle behavior
- Fixture-based server lifecycle management ensures deterministic test execution and automatic resource cleanup

Negative:
- Dependency on trustme library introduces external test infrastructure dependency that must be maintained
- Parameterized fixtures increase test execution time by running each test across multiple network protocol variants
- Socket-level TLS version inspection couples tests to low-level implementation details that may vary across platforms
- IPv6 test skipping logic adds conditional complexity and may mask platform-specific compatibility issues

## Alternatives

- Use pre-generated static certificates committed to repository (rejected)
  Rejected because: Static certificates expire, require manual rotation, and cannot dynamically adapt to parameterized host configurations (localhost vs 127.0.0.1 vs ::1)
  When valid: When certificate configuration is fixed and test scenarios do not require dynamic SAN or common name variations
- Mock TLS connections instead of running real HTTPS servers (rejected)
  Rejected because: Mocking would not validate actual TLS version negotiation, certificate validation, or socket-level behavior that the evidence shows is being tested
  When valid: When testing application logic that does not depend on actual TLS handshake behavior or certificate validation
- Use unittest.TestCase class-based fixtures instead of pytest fixtures (rejected)
  Rejected because: unittest lacks native parameterization support and would require manual test duplication for loopback host variants
  When valid: When pytest is not available or when test suite must remain framework-agnostic

## Risks

- Platform-specific IPv6 availability may cause test skips that mask actual compatibility issues
  Mitigation: Ensure CI environments enable IPv6 support and monitor skip rates to detect platform configuration drift
  Owner: engineering team
- Socket-level TLS version detection may behave inconsistently across OpenSSL versions or Python SSL module implementations
  Mitigation: Document minimum supported OpenSSL and Python versions, add version detection to test setup validation
  Owner: engineering team
- Connection pool state inspection through mutable collections may introduce test ordering dependencies if not properly isolated
  Mitigation: Ensure each test creates fresh connection pool instances and verify fixture scope prevents state leakage
  Owner: engineering team

## Implementation Notes

- Use pytest.TempPathFactory.mktemp() to create isolated certificate directories for each test fixture invocation
- Implement HAS_IPV6 platform detection early in test suite initialization to provide clear skip messages for IPv6 tests
- Wrap server lifecycle in context managers (run_server_in_thread) to guarantee cleanup even when tests fail or raise exceptions
- Initialize observable collections (tls_versions set, connections set) within fixture scope to ensure test isolation and prevent cross-test contamination

## Continuation Context


Verify commands:
- grep -r '@pytest.fixture.*params.*localhost.*127\.0\.0\.1.*::1' test/
- grep -r 'trustme\.CA\(\)' test/
- grep -r 'tls_versions\.add\(_sock\.version\(\)\)' test/
- grep -r 'connections\.add\(conn\)' test/

Accept when:
- All grep commands return matches in test/conftest.py and test/test_poolmanager.py confirming parameterized fixtures, trustme CA usage, and data access patterns
- Test suite executes successfully across all parameterized loopback host variants (localhost, 127.0.0.1, ::1) on IPv6-enabled platforms
- Connection pool tests demonstrate observable connection reuse through collection-based tracking patterns

## Enforcement

- Verified by: CI pipeline executes pytest test suite with coverage reporting
- Verified by: Code review verifies new test fixtures follow parameterization and context manager patterns
- Verified by: Static analysis checks for proper fixture scope and cleanup patterns
- Violation handling: CI build fails if test fixtures do not properly clean up server resources
- Violation handling: Code review blocks merge if new TLS tests bypass trustme CA infrastructure
- Violation handling: Test failures on IPv6-enabled platforms trigger investigation of platform compatibility
- Exception process: Platform-specific test skips require documentation in test docstrings explaining why IPv6 or specific TLS versions are unavailable
- Exception process: Alternative certificate generation approaches require architecture review and justification for deviation from trustme pattern
- Exception process: Connection pool testing exceptions require demonstration that observable state tracking is infeasible for specific scenarios
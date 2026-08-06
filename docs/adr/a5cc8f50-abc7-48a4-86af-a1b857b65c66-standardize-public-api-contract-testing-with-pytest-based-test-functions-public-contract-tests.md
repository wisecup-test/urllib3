# Standardize Public API Contract Testing with Pytest-Based Test Functions: Public Contract Tests

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 27 test files with explicit public API contract testing functions, primarily using pytest as the test framework with test functions following the test_* naming convention
- Test modules systematically validate public interfaces including proxy management (test_proxy_headers, test_default_port), file posting (test_input_datastructures, test_field_encoding), connection handling (test_match_hostname_*), and HTTP/2 operations (test_putheader, test_request_putheader)
- The urllib3 library serves as the primary system under test, with tests covering core modules including urllib3.poolmanager, urllib3.connectionpool, urllib3.connection, urllib3.fields, urllib3.filepost, and urllib3.exceptions
- Integration and socket-level testing demonstrates contract validation at multiple abstraction layers, from unit tests of individual functions to socket-level protocol verification with raw HTTP message handling
- The test infrastructure includes fixture-based configuration (pytest.fixture), parameterized tests, and specialized test utilities for SSL/TLS contexts, certificate management (trustme.CA), and server simulation

## Problem Statement

Without a standardized approach to public API contract testing, the codebase risks inconsistent validation of interface boundaries, incomplete coverage of public methods and functions, and difficulty maintaining backward compatibility guarantees as the urllib3 library evolves across versions.

## Decision

1. MUST: Public API contract tests MUST validate both successful execution paths and error conditions using pytest.raises for exception verification

## Policy Block

- MUST Public API contract tests MUST validate both successful execution paths and error conditions using pytest.raises for exception verification

In scope:
- All public functions, methods, and classes exported from urllib3 modules including poolmanager, connectionpool, connection, fields, filepost, and exceptions
- Public API contracts for HTTP client operations including request/response handling, connection pooling, proxy configuration, and SSL/TLS operations
- Integration points between urllib3 and external systems including socket-level protocol handling and HTTP message formatting
- Error handling contracts including exception types, error messages, and failure modes exposed through public interfaces

Out of scope:
- Internal implementation details not exposed through public APIs
- Performance benchmarking and load testing (covered under separate performance testing strategy)
- Security vulnerability scanning and penetration testing (covered under separate security testing strategy)
- Documentation generation and API reference validation

Exceptions:
- EXC-001: Legacy test modules written before pytest adoption may use unittest.TestCase-based test classes
- EXC-002: Platform-specific or environment-specific tests may skip contract validation using pytest.skip when required dependencies are unavailable

## Rationale

- The evidence shows 27 files with consistent pytest-based contract testing patterns, demonstrating an established practice with 91.95% confidence across the codebase
- Systematic validation of public API contracts through dedicated test functions provides regression protection and enables confident refactoring of internal implementations without breaking external consumers
- The pytest framework provides superior fixture management, parameterization, and exception handling compared to alternatives, as evidenced by widespread adoption across test modules including conftest.py fixture definitions
- Multi-layer testing from unit to socket-level integration ensures contracts are validated at appropriate abstraction levels, catching both logical errors and protocol-level implementation issues

## Consequences

Positive:
- Consistent test organization and naming conventions improve test discoverability and maintainability across the 27+ test modules
- Explicit contract validation through dedicated test functions provides clear documentation of expected API behavior and backward compatibility guarantees
- Pytest fixture-based test infrastructure enables efficient test resource management and reduces test setup duplication
- Multi-layer testing strategy catches contract violations at both unit and integration levels, improving overall API reliability

Negative:
- Maintaining comprehensive contract tests requires ongoing effort as new public APIs are added or existing APIs evolve
- Socket-level integration tests introduce complexity in test infrastructure including mock servers, certificate management, and protocol simulation
- Test execution time increases with comprehensive contract coverage, potentially slowing CI/CD pipeline feedback cycles
- Pytest-specific features create framework lock-in, making future test framework migrations more costly

## Alternatives

- Use unittest.TestCase-based test classes exclusively without pytest fixtures or parameterization (rejected)
  Rejected because: Evidence shows pytest adoption across 27 files with extensive use of fixtures (conftest.py), parameterization, and pytest.raises, indicating unittest.TestCase would require abandoning established infrastructure and losing superior test organization capabilities
  When valid: May be appropriate for isolated test modules with no fixture dependencies or parameterization requirements
- Implement contract testing through property-based testing using hypothesis for generative test cases (deferred)
  Rejected because: No evidence of hypothesis usage in the detected patterns; property-based testing could complement but not replace explicit contract validation for critical API boundaries
  When valid: Could be adopted for specific API contracts with complex input spaces where example-based testing provides insufficient coverage
- Generate contract tests automatically from API specifications or type annotations (rejected)
  Rejected because: Evidence shows manually written test functions with explicit assertions tailored to specific contract requirements; automated generation would lose domain-specific validation logic and edge case coverage
  When valid: May provide baseline coverage for simple CRUD operations or REST API endpoints with OpenAPI specifications

## Risks

- Contract tests may become outdated as APIs evolve, leading to false confidence in backward compatibility
  Mitigation: Implement CI checks that fail when public API changes are detected without corresponding test updates; require test updates in same commit as API changes
  Owner: Engineering team and test infrastructure maintainers
- Socket-level integration tests may exhibit flakiness due to timing issues, port conflicts, or platform-specific networking behavior
  Mitigation: Use pytest fixtures for proper resource cleanup, implement retry logic for transient failures, and leverage pytest.skip for platform-specific test exclusion as evidenced in conftest.py
  Owner: Test infrastructure team
- Incomplete contract coverage may leave critical API boundaries unvalidated despite comprehensive test suite
  Mitigation: Implement coverage tracking specifically for public API functions and methods; require contract tests for all new public APIs during code review
  Owner: Engineering team and code reviewers

## Implementation Notes

- Organize test modules by functional domain matching the source module structure (e.g., test_connectionpool.py tests src/urllib3/connectionpool.py)
- Use conftest.py for shared fixtures including server configuration, certificate generation (trustme.CA), and loopback host parameterization as demonstrated in the evidence
- Implement socket-level integration tests using handler functions (e.g., multicookie_response_handler, socket_handler) that simulate protocol-level interactions
- Leverage pytest.raises context manager for exception contract validation and pytest.skip for platform-specific or environment-dependent test exclusion
- Follow test function naming convention test_<contract_being_validated> with descriptive names that document the specific API behavior under test

## Continuation Context


Verify commands:
- grep -r "^def test_" test/ | wc -l
- pytest --collect-only -q test/ | grep "<Function" | wc -l
- grep -r "@pytest.fixture" test/conftest.py | wc -l
- pytest test/ -v --tb=short -k "test_proxy or test_connection or test_filepost"

Accept when:
- All test modules in test/ directory contain test functions following the test_* naming convention and use pytest as the test framework
- Public API contracts for core urllib3 modules (poolmanager, connectionpool, connection, fields, filepost) have corresponding test modules with dedicated contract validation functions
- Test execution via pytest successfully discovers and runs contract tests with clear pass/fail results for each validated API boundary

## Enforcement

- Verified by: CI pipeline pytest execution with mandatory test pass requirement before merge
- Verified by: Code review verification that new public APIs include corresponding contract tests
- Verified by: Test coverage reporting with specific tracking of public API function coverage
- Violation handling: CI build failure blocks merge when contract tests fail or are missing for modified public APIs
- Violation handling: Code review rejection for pull requests that add or modify public APIs without corresponding test updates
- Violation handling: Automated coverage reports flag public API functions lacking contract test coverage
- Exception process: Exception requests must document specific rationale for skipping contract tests (e.g., platform limitations, external dependency unavailability)
- Exception process: Test module maintainer approval required for unittest.TestCase-based tests in new modules
- Exception process: Platform-specific test skips using pytest.skip are automatically approved when runtime environment detection justifies the skip
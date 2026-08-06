# Establish HTTP Client Compatibility Testing for Public API Contracts: Test Functions Follow

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains compatibility testing infrastructure in test/test_compatibility.py that validates HTTP client behavior across multiple protocol versions and library implementations
- Evidence shows explicit testing of urllib3.http2, http.cookiejar, urllib, and pytest frameworks with test functions test_extract and test_h2_version_check
- The test suite validates public API contracts through urllib.request.Request instantiation and mock/patch boundary definitions
- Service boundary testing is implemented to ensure consistent behavior across different HTTP client libraries and protocol versions (HTTP/2 compatibility checks)
- The pattern emerged from the need to maintain stable public API contracts while supporting multiple HTTP client implementations and protocol versions

## Problem Statement

Without systematic compatibility testing of HTTP client libraries and protocol versions, public API contracts may break silently when dependencies are upgraded or when clients use different HTTP implementations, leading to runtime failures and integration issues that are difficult to diagnose.

## Decision

1. SHOULD: Test functions SHOULD follow naming convention test_<feature> for discoverability and automated test execution

## Policy Block

- SHOULD Test functions SHOULD follow naming convention test_<feature> for discoverability and automated test execution

In scope:
- All public-facing HTTP API endpoints and contracts
- HTTP client library integrations (urllib, urllib3, http.cookiejar)
- HTTP protocol version compatibility (HTTP/1.1, HTTP/2)
- Service boundary mocking and isolation testing

Out of scope:
- Internal service-to-service communication not exposed through public APIs
- Non-HTTP protocols (WebSocket, gRPC) unless explicitly added to compatibility suite
- Performance testing or load testing of HTTP clients
- Security testing of HTTP implementations (covered by separate security ADRs)

## Rationale

- The evidence shows explicit compatibility testing infrastructure with test_extract and test_h2_version_check functions, indicating a deliberate pattern of validating API contracts across implementations
- Multiple HTTP client libraries (urllib, urllib3, http.cookiejar) are tested together, demonstrating the need to support diverse client ecosystems
- HTTP/2 version checking indicates protocol-level compatibility requirements that must be validated to prevent runtime failures
- Mock and patch usage for service boundaries shows architectural intent to isolate and test API contracts independently of implementation details

## Consequences

Positive:
- Early detection of breaking changes when HTTP client libraries or protocol versions are updated
- Increased confidence in public API stability across different client implementations
- Clear documentation of supported HTTP client libraries and protocol versions through executable tests
- Reduced integration issues for API consumers using different HTTP client stacks

Negative:
- Increased test maintenance burden when new HTTP client libraries or protocol versions need to be supported
- Longer test execution time due to multiple client implementation validations
- Potential for test brittleness if mocking strategies are not carefully maintained
- Additional complexity in CI/CD pipelines to manage multiple HTTP client dependencies

## Alternatives

- Test only against a single canonical HTTP client implementation (e.g., urllib3 only) (rejected)
  Rejected because: Evidence shows multiple client libraries are already in use (urllib, urllib3, http.cookiejar), and limiting to one would not catch compatibility issues affecting real-world API consumers
  When valid: In greenfield projects with strict control over client library choices and no legacy compatibility requirements
- Rely on integration tests in production or staging environments to catch compatibility issues (rejected)
  Rejected because: Post-deployment detection of compatibility issues is too late and costly; the existing test_compatibility.py pattern enables pre-deployment validation
  When valid: Never recommended for public API contracts where breaking changes have external impact
- Use contract testing frameworks (Pact, Spring Cloud Contract) instead of direct HTTP client testing (deferred)
  Rejected because: Current evidence shows direct testing approach is working; contract testing frameworks could be evaluated as the API ecosystem grows
  When valid: When managing complex multi-service ecosystems with many consumer teams requiring formal contract guarantees

## Risks

- Test coverage may not include all HTTP client libraries actually used by API consumers in production
  Mitigation: Monitor API access logs to identify client libraries in use and add them to compatibility test suite; document supported clients in API documentation
  Owner: API Platform Team
- HTTP/2 compatibility tests may pass but fail in production due to infrastructure differences (proxies, load balancers)
  Mitigation: Supplement unit tests with integration tests in staging environments that mirror production infrastructure; include infrastructure components in compatibility validation
  Owner: DevOps and API Platform Teams
- Mock-based service boundary tests may diverge from actual service behavior over time
  Mitigation: Regularly validate mocks against real service responses; implement contract tests that verify mock accuracy; schedule periodic mock refresh cycles
  Owner: Engineering Team

## Implementation Notes

- Place all compatibility tests in test/test_compatibility.py following the established pattern with test_extract and test_h2_version_check as examples
- Use pytest fixtures to parameterize tests across multiple HTTP client implementations, reducing code duplication
- Implement TestCookiejar and TestInitialization classes to organize related compatibility test cases by functional area
- Use unittest.mock.patch for service boundary isolation, ensuring tests remain fast and deterministic without external dependencies
- Document supported HTTP client libraries and protocol versions in API documentation, referencing compatibility test coverage

## Continuation Context


Verify commands:
- grep -r "def test_.*compatibility" test/ || grep -r "test_h2_version_check\|test_extract" test/test_compatibility.py
- python -m pytest test/test_compatibility.py -v --collect-only | grep -E "test_extract|test_h2_version_check"
- grep -E "urllib3\.http2|http\.cookiejar|urllib" test/test_compatibility.py

Accept when:
- test/test_compatibility.py exists and contains test functions for HTTP client compatibility validation
- Tests validate at least urllib, urllib3, and http.cookiejar client implementations
- HTTP/2 version compatibility is explicitly tested through dedicated test functions
- All compatibility tests pass in CI pipeline before deployment

## Enforcement

- Verified by: Automated pytest execution in CI pipeline for test/test_compatibility.py
- Verified by: Code review verification that new public API endpoints include corresponding compatibility tests
- Verified by: Test coverage reports showing compatibility test execution for API contract changes
- Violation handling: CI pipeline fails if compatibility tests are missing for new public API endpoints
- Violation handling: Pull requests adding or modifying public APIs require compatibility test updates before merge approval
- Violation handling: Quarterly audit of API endpoints to ensure compatibility test coverage is complete
- Exception process: Exception requests must document why compatibility testing is not applicable for specific API endpoints
- Exception process: API Platform Team lead must approve exceptions with written justification
- Exception process: Exceptions are reviewed quarterly and must be re-justified or remediated
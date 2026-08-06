# Adopt Safe Dictionary Get Pattern for External Client Response Headers: Test Code Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The urllib3 library interacts with external HTTP clients and servers where response headers may be missing, malformed, or inconsistent across implementations
- Production code in connectionpool.py and retry.py uses .get() with defaults when accessing response headers like 'Retry-After' to handle missing headers gracefully
- Test code across multiple test modules (test_poolmanager.py, test_proxy_poolmanager.py, test_response.py) consistently uses .get() when validating headers returned from dummy servers
- The pattern appears in 11 files with 91.71% confidence, indicating systematic adoption rather than isolated usage
- External client boundaries require defensive programming because header presence and format cannot be guaranteed by the library

## Problem Statement

When testing HTTP client libraries that interact with external servers, direct dictionary access to response headers (e.g., headers['Location']) raises KeyError exceptions when headers are absent, causing test failures and production errors. This creates brittle integration points that fail unpredictably based on server implementation variations.

## Decision

1. MUST: Test code MUST use .get() method when validating headers in integration tests with dummy servers or external endpoints

## Policy Block

- MUST Test code MUST use .get() method when validating headers in integration tests with dummy servers or external endpoints

In scope:
- HTTP response header access in production code (connectionpool.py, retry.py, response.py)
- Integration test header validation (test_poolmanager.py, test_proxy_poolmanager.py, test_response.py)
- Environment variable access at external boundaries (os.environ.get)
- Port and scheme lookups from external configuration (port_by_scheme.get)

Out of scope:
- Internal data structure access where key presence is guaranteed by construction
- Dictionary access within private methods where preconditions ensure key existence
- Configuration dictionaries loaded and validated at startup

## Rationale

- Evidence shows consistent .get() usage across 11 files spanning production code (connectionpool.py, retry.py, ssl_.py) and test code (test_poolmanager.py, test_proxy_poolmanager.py, test_response.py)
- The pattern appears specifically at external client boundaries: response.headers.get('Retry-After'), returned_headers.get('Location'), os.environ.get('CI'), demonstrating defensive programming at trust boundaries
- High confidence (91.71%) and support count (11 files) indicate this is an established architectural pattern rather than incidental usage
- The pattern enables graceful degradation when external clients provide incomplete or non-standard responses, improving reliability in production

## Consequences

Positive:
- Eliminates KeyError exceptions when external servers omit expected headers, improving production stability
- Test code becomes more resilient to variations in dummy server implementations and external API changes
- Enables graceful fallback behavior (e.g., default ports, retry logic) when optional headers are missing
- Reduces coupling between client code and specific server implementations by not assuming header presence

Negative:
- Silent failures may occur if code does not properly handle None returns from .get() calls
- Debugging becomes harder when missing headers are silently replaced with defaults rather than raising exceptions
- Increased verbosity in code compared to direct dictionary access
- May mask configuration errors or API contract violations that should be caught early

## Alternatives

- Use direct dictionary access (headers['Location']) and catch KeyError exceptions at call sites (rejected)
  Rejected because: Creates verbose exception handling at every access point and makes code harder to read; evidence shows .get() is preferred pattern across codebase
  When valid: When header presence is guaranteed by API contract and absence indicates a critical error that should halt execution
- Create wrapper classes with property accessors that provide defaults (rejected)
  Rejected because: Adds complexity and indirection; evidence shows direct .get() usage is sufficient and widely adopted
  When valid: When building a higher-level API abstraction over raw HTTP responses with complex validation logic
- Use schema validation libraries (e.g., pydantic) to validate response structure (deferred)
  Rejected because: Not evident in current codebase; would require significant refactoring
  When valid: For new services or major refactoring where strict response validation is required

## Risks

- Silent failures when None returns from .get() are not properly handled, leading to AttributeError or TypeError downstream
  Mitigation: Add type hints and static analysis (mypy) to catch None handling issues; include explicit None checks in critical paths
  Owner: engineering team
- Inconsistent application of pattern where some code uses .get() and other code uses direct access, creating maintenance confusion
  Mitigation: Add linting rules to detect direct dictionary access at external boundaries; document pattern in contribution guidelines
  Owner: engineering team
- Default values may not be appropriate for all contexts, leading to incorrect behavior
  Mitigation: Review default values in code review; prefer explicit defaults over None where possible; document expected behavior when headers are missing
  Owner: engineering team

## Implementation Notes

- Use response.headers.get('Header-Name') for all HTTP response header access in production code
- Provide explicit defaults when appropriate: port_by_scheme.get(scheme, 80) rather than relying on None
- In test code, use assertions that handle None: assert returned_headers.get('Foo') == expected_value or returned_headers.get('Foo') is None
- Document expected behavior when headers are missing in function docstrings, especially for retry logic and connection pooling

## Continuation Context


Verify commands:
- grep -r '\.headers\[' src/urllib3/ test/ --include='*.py' | grep -v '.get(' | wc -l
- grep -r 'response\.headers\.get\|returned_headers\.get\|os\.environ\.get' src/urllib3/ test/ --include='*.py' | wc -l
- python -m pytest test/test_connectionpool.py test/with_dummyserver/test_poolmanager.py -v

Accept when:
- First verify command returns 0 (no direct dictionary access to headers in production or test code)
- Second verify command returns >50 (widespread .get() usage at external boundaries)
- All integration tests pass without KeyError exceptions from header access

## Enforcement

- Verified by: Code review checklist requiring .get() usage for external client boundaries
- Verified by: Static analysis with custom linting rules detecting direct dictionary access to response.headers
- Verified by: Integration test suite execution in CI validating header access patterns
- Violation handling: CI build fails if linting rules detect direct dictionary access at external boundaries
- Violation handling: Code review blocks merge if .get() pattern is not followed for response headers
- Violation handling: Production monitoring alerts on KeyError exceptions from header access
- Exception process: Document justification in code comments explaining why direct access is safe
- Exception process: Obtain approval from two reviewers familiar with the external client contract
- Exception process: Add explicit KeyError handling with logging if direct access is required
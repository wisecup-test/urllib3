# Use .get() Method with Default Values for Safe External Header and Environment Access: Access Ssl Certificate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase interacts with external HTTP clients through response headers, request headers, and environment variables that may or may not be present
- HTTP headers from external clients (Retry-After, Location, Content-Type, Foo, Baz, Host, etc.) are accessed across connection pools, response handlers, and proxy managers
- Environment variables (CI, GITHUB_ACTIONS, SSLKEYLOGFILE, READTHEDOCS_CANONICAL_URL) are read from external runtime contexts
- Dictionary-like objects (headers, environ, cert dictionaries) require safe access patterns to prevent KeyError exceptions when keys are absent
- The pattern appears in 14 files across core urllib3 modules, test suites, and dummy server implementations with 91.46% confidence

## Problem Statement

External clients and runtime environments provide unpredictable key-value data structures where keys may be absent, requiring a defensive access pattern that prevents runtime exceptions while providing sensible defaults for missing values.

## Decision

1. MUST: All access to SSL certificate dictionaries (subjectAltName, subject) MUST use .get() with empty tuple or appropriate default

## Policy Block

- MUST All access to SSL certificate dictionaries (subjectAltName, subject) MUST use .get() with empty tuple or appropriate default

In scope:
- HTTP response headers received from external servers
- HTTP request headers received from external clients
- Environment variables read from os.environ
- SSL certificate dictionaries from external TLS handshakes
- Configuration lookup tables with optional keys

Out of scope:
- Internal data structures with guaranteed schema
- Dataclass fields with required attributes
- Function parameters with explicit type contracts

Exceptions:
- EXC-001: The code explicitly validates key presence before access and handles KeyError in an exception handler

## Rationale

- Evidence shows consistent use of .get() across 14 files for headers (response.headers.get('Retry-After'), returned_headers.get('Foo')), environment variables (os.environ.get('CI')), and certificate fields (cert.get('subjectAltName', ()))
- The pattern prevents KeyError exceptions when external clients omit optional headers or when runtime environments lack expected variables
- Default values enable graceful degradation: missing Retry-After headers default to None, missing environment flags default to False, missing cert fields default to empty tuples
- The 91.46% confidence across urllib3 core modules (connectionpool.py, fields.py, retry.py, ssl_.py) and test infrastructure indicates an established architectural convention

## Consequences

Positive:
- Eliminates KeyError exceptions when external clients send incomplete or non-standard HTTP headers
- Enables safe feature detection and optional behavior based on environment variable presence
- Provides predictable fallback behavior for missing configuration values
- Simplifies error handling by converting absence into default values rather than exceptions

Negative:
- Silent failures may occur if default values mask configuration errors or missing required data
- Type inconsistency risk when default values do not match expected types for present values
- Debugging difficulty when missing keys are silently replaced with defaults rather than raising explicit errors
- Potential for incorrect behavior if downstream code does not properly handle default sentinel values

## Alternatives

- Use direct dictionary access with try/except KeyError blocks (rejected)
  Rejected because: Verbose and clutters code with exception handling boilerplate; evidence shows no usage of this pattern in the 14 detected files
  When valid: When KeyError itself carries semantic meaning that must be logged or handled distinctly
- Use collections.defaultdict for all external data structures (rejected)
  Rejected because: Requires wrapping external objects (HTTP headers, os.environ) which are provided by libraries and cannot be replaced; adds unnecessary abstraction layer
  When valid: For internal data structures fully under application control
- Validate all required keys upfront and fail fast (deferred)
  Rejected because: Not applicable for optional headers and environment variables where absence is valid; would break backward compatibility
  When valid: For truly required fields where absence indicates a protocol violation or misconfiguration

## Risks

- Default values may mask genuine configuration errors or missing required data, leading to silent failures in production
  Mitigation: Document which fields are truly optional vs. required; add logging at INFO level when critical fields use defaults; validate required fields explicitly before .get() access
  Owner: Engineering team
- Type mismatches between default values and actual values may cause downstream type errors
  Mitigation: Use type hints to document expected types; ensure default values match the type of present values; add runtime type validation for critical paths
  Owner: Engineering team
- Inconsistent default values across the codebase may lead to unpredictable behavior
  Mitigation: Document standard default values for common headers and environment variables; create constants for frequently used defaults; enforce through code review
  Owner: Engineering team

## Implementation Notes

- For HTTP headers, use None as default when absence is semantically meaningful (e.g., response.headers.get('Retry-After') returns None to indicate no retry guidance)
- For environment variables, use empty string or False as default based on downstream boolean vs. string usage (e.g., os.environ.get('CI') for boolean checks)
- For certificate fields, use empty tuple () as default to enable safe iteration (e.g., cert.get('subjectAltName', ()))
- For lookup tables and mappings, provide fallback values that maintain type consistency (e.g., port_by_scheme.get(scheme, 80) for integer ports)

## Continuation Context


Verify commands:
- grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal
- grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l  # Should be 0 or minimal
- grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l  # Should match pattern usage

Accept when:
- All HTTP header access in src/urllib3/ uses .get() method with appropriate defaults
- All os.environ access uses .get() method with appropriate defaults
- No KeyError exceptions occur in production logs related to header or environment variable access

## Enforcement

- Verified by: Code review checklist requiring .get() usage for external data access
- Verified by: Static analysis with grep patterns in CI pipeline checking for direct dictionary access to headers and environ
- Verified by: Unit tests validating behavior when optional headers and environment variables are absent
- Violation handling: CI pipeline fails if direct dictionary access patterns are detected in new code
- Violation handling: Code review blocks merge if external data structures use direct access without documented justification
- Violation handling: Runtime monitoring alerts on KeyError exceptions from header or environment access
- Exception process: Document the specific case where direct access is required in code comments
- Exception process: Obtain maintainer approval in pull request review
- Exception process: Add explicit KeyError handling or validation logic to justify the exception
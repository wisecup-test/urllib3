# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Logger Instances Stored

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses Python's standard logging module with module-level logger initialization via logging.getLogger(__name__) across multiple components including connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Logger initialization follows a consistent pattern where each module obtains its own logger instance using the __name__ variable, enabling hierarchical logging namespaces that reflect the module structure
- The logging infrastructure supports integration testing scenarios where components interact with external boundaries (response.headers.get, port_by_scheme.get) and internal state management (self.pool.put, self.headers.get)
- The pattern appears in modules handling connection pooling, retry logic, and HTTP response processing, indicating logging serves both operational observability and debugging needs across network I/O boundaries

## Problem Statement

Without a standardized approach to logger initialization, modules may use inconsistent logging patterns (print statements, ad-hoc logger names, or shared logger instances), making it difficult to filter, route, and control log output granularity during testing, debugging, and production monitoring. This inconsistency complicates integration testing where observability into component behavior at external boundaries is essential for diagnosing failures.

## Decision

1. SHOULD: Logger instances SHOULD be stored in a module-level variable named 'log' or 'logger' for consistency

## Policy Block

- SHOULD Logger instances SHOULD be stored in a module-level variable named 'log' or 'logger' for consistency

In scope:
- All Python modules in src/ directory
- Connection pooling components (connectionpool.py)
- Retry logic and backoff mechanisms (util/retry.py)
- HTTP response processing modules (contrib/emscripten/response.py)
- Any module requiring observability during integration testing

Out of scope:
- Test fixtures and test helper modules (may use custom logging configurations)
- Third-party library code
- Vendored dependencies
- Build scripts and tooling

## Rationale

- The pattern is observed consistently across 3 files with 91.60% confidence, indicating an established convention rather than isolated usage
- Using __name__ for logger initialization creates hierarchical namespaces (e.g., urllib3.connectionpool, urllib3.util.retry) that enable fine-grained log filtering during testing and production debugging
- The co-occurrence of logging with integration testing patterns (response.headers.get, self.pool.put) demonstrates that logging serves as a testing strategy for observing behavior at external boundaries
- Standardized logger initialization supports test isolation by allowing per-module log level configuration without affecting other components

## Consequences

Positive:
- Enables hierarchical log filtering by module path, allowing developers to focus on specific components during integration testing
- Provides consistent observability across connection pooling, retry logic, and response processing without coupling to specific logging backends
- Supports test-time log capture and assertion by providing predictable logger names based on module structure
- Facilitates debugging of integration boundaries where external clients (headers.get, port_by_scheme.get) interact with internal state

Negative:
- Requires explicit import of logging module in every file that needs logging, increasing boilerplate
- Module-level logger initialization occurs at import time, which may complicate testing scenarios that need to mock or reconfigure logging before module import
- Logger namespace hierarchy is tightly coupled to module structure, making refactoring more complex if modules are renamed or moved
- Does not enforce log level consistency across modules, potentially leading to verbose output in some components and silence in others

## Alternatives

- Use a single shared logger instance imported from a central logging module (rejected)
  Rejected because: Eliminates hierarchical namespace benefits and makes it impossible to filter logs by module during testing; observed evidence shows per-module logger pattern is already established
  When valid: Only appropriate for very small codebases with fewer than 5 modules where log filtering is not required
- Use print statements or sys.stderr.write for debugging output (rejected)
  Rejected because: Provides no filtering, routing, or level control; cannot be captured by test frameworks; does not support production observability requirements
  When valid: Never valid for production code; acceptable only in throwaway debugging scripts
- Lazy logger initialization within functions or methods rather than module level (deferred)
  Rejected because: Adds runtime overhead and complexity; however, may be necessary for modules with circular import issues
  When valid: When module-level initialization causes circular import problems that cannot be resolved through restructuring

## Risks

- Module-level logger initialization may cause issues in test environments that need to configure logging before module import
  Mitigation: Document test setup patterns that configure logging handlers before importing application modules; provide test fixtures that reset logging configuration between tests
  Owner: Testing infrastructure team
- Inconsistent log levels across modules may result in either excessive verbosity or insufficient observability during integration testing
  Mitigation: Establish default log level configuration in test fixtures; document recommended log levels for each module category (connection pooling, retry logic, response processing)
  Owner: Engineering team
- Logger namespace coupling to module paths may complicate large-scale refactoring efforts
  Mitigation: Include logger name updates in refactoring checklists; use grep-based verification to ensure logger names remain consistent with module structure
  Owner: Engineering team

## Implementation Notes

- Add 'import logging' and 'logger = logging.getLogger(__name__)' at the top of each new Python module after imports
- For existing modules without logging, add logger initialization before first function or class definition
- In integration tests, configure log capture using pytest's caplog fixture or unittest's assertLogs context manager to verify behavior at external boundaries
- Use logger.debug() for detailed state transitions, logger.info() for integration boundary crossings, logger.warning() for retry attempts, and logger.error() for failures

## Continuation Context


Verify commands:
- grep -r 'logging\.getLogger(__name__)' src/ | wc -l
- grep -r 'import logging' src/ | wc -l
- python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20

Accept when:
- All Python modules in src/ directory that require observability contain 'logging.getLogger(__name__)' initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names

## Enforcement

- Verified by: Code review checklist requiring logger initialization in new modules
- Verified by: Grep-based verification in CI pipeline checking for logging.getLogger(__name__) pattern
- Verified by: Integration test suite validation that logger names match module structure
- Violation handling: CI pipeline fails if new modules lack proper logger initialization
- Violation handling: Code review feedback requests addition of logging.getLogger(__name__) before merge
- Violation handling: Linting rules flag hardcoded logger names or shared logger instances
- Exception process: Document exception rationale in module docstring if module-level initialization is not possible
- Exception process: Obtain approval from testing infrastructure team for alternative logging patterns
- Exception process: Add exception to .pylintrc or similar configuration with explanatory comment
# Standardize Logging at Message Queue Boundaries: Modules Implementing Protocol

Status: proposed
Date: 2025-01-20
Deciders: Detection Pipeline (automated)

## Context

- The codebase exhibits a pattern where logging infrastructure (logging.getLogger(__name__)) is consistently initialized in modules that perform message queue operations via send() methods
- Two distinct protocol implementations (HTTP/2 connection handling and PyOpenSSL socket wrapping) both demonstrate this logging-at-boundary pattern when transmitting data
- The pattern appears in modules that bridge between high-level API contracts (connect, putrequest, putheader, endheaders, send) and low-level transmission boundaries (self.send(body), self.connection.send(data))
- Both implementations use Python's standard logging module with module-level loggers, suggesting a deliberate architectural choice for observability at I/O boundaries

## Problem Statement

When data crosses message queue or transmission boundaries in protocol implementations, there is a need for consistent observability to diagnose connection issues, data transmission failures, and protocol-level errors without requiring invasive debugging or instrumentation changes.

## Decision

1. SHOULD: Modules implementing protocol-level send operations SHOULD log at appropriate levels (DEBUG for routine operations, WARNING for recoverable issues, ERROR for failures)

## Policy Block

- SHOULD Modules implementing protocol-level send operations SHOULD log at appropriate levels (DEBUG for routine operations, WARNING for recoverable issues, ERROR for failures)

In scope:
- Modules implementing HTTP/2 connection handling
- Modules implementing SSL/TLS socket wrapping
- Any module that defines send() methods for data transmission
- Protocol adapter implementations that bridge API contracts to low-level I/O

Out of scope:
- Pure data transformation modules without I/O operations
- Configuration or initialization modules
- Test fixtures and mock implementations
- Modules that only consume logging without implementing transmission boundaries

## Rationale

- The pattern is observed in 2 files with 92.50% confidence, both implementing critical protocol boundaries (HTTP/2 and SSL/TLS) where observability is essential for production debugging
- Consistent use of logging.getLogger(__name__) provides namespace isolation and enables fine-grained log level control per module without code changes
- The co-occurrence of logging infrastructure with send() methods at message queue boundaries indicates a deliberate architectural decision to instrument data transmission points
- This pattern enables post-mortem analysis and real-time monitoring of protocol-level operations without requiring debugger attachment or code modification

## Consequences

Positive:
- Consistent observability across all protocol implementations and transmission boundaries
- Module-level logger naming enables targeted log level configuration in production without code changes
- Reduced time-to-diagnosis for connection failures, protocol errors, and data transmission issues
- Standardized logging pattern reduces cognitive load for developers working across different protocol implementations

Negative:
- Additional runtime overhead from logging infrastructure initialization and log statement evaluation
- Potential for excessive log volume if DEBUG level is enabled globally rather than per-module
- Risk of logging sensitive data (credentials, tokens, PII) if transmission payloads are logged without sanitization
- Maintenance burden to ensure logging statements remain useful and don't become stale as code evolves

## Alternatives

- Use centralized tracing/instrumentation framework (OpenTelemetry, Jaeger) instead of logging (rejected)
  Rejected because: Introduces external dependencies and complexity; logging provides sufficient observability for the current use case with zero external dependencies
  When valid: When distributed tracing across multiple services is required or when correlation of requests across system boundaries is needed
- Implement custom debug hooks or callbacks for transmission events (rejected)
  Rejected because: Requires custom infrastructure and API surface; Python's logging module is standardized, well-understood, and integrates with existing tooling
  When valid: When programmatic access to transmission events is needed for testing or when logging overhead is prohibitive
- No logging at transmission boundaries, rely on network-level monitoring (rejected)
  Rejected because: Network-level monitoring lacks application context (protocol state, API call context, error conditions); cannot diagnose application-level issues
  When valid: In extremely performance-sensitive contexts where even logging overhead is unacceptable, or when network infrastructure provides sufficient observability

## Risks

- Sensitive data (authentication tokens, PII, credentials) may be inadvertently logged at transmission boundaries
  Mitigation: Implement log sanitization utilities and code review guidelines to prevent logging of sensitive payloads; use structured logging with explicit field inclusion rather than dumping entire payloads
  Owner: Engineering team
- Excessive logging at high-throughput boundaries may impact performance or generate unsustainable log volumes
  Mitigation: Use DEBUG level for routine operations; implement sampling or rate-limiting for high-frequency events; monitor log volume and performance impact in production
  Owner: Engineering team
- Inconsistent logging practices across modules may reduce the value of standardization
  Mitigation: Document logging standards in developer guidelines; use linting or static analysis to detect missing loggers in transmission boundary modules; include logging review in code review checklist
  Owner: Engineering team

## Implementation Notes

- Initialize the module-level logger immediately after imports: `logger = logging.getLogger(__name__)` to ensure it's available throughout the module
- For send() methods, log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions
- Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled
- Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems

## Continuation Context


Verify commands:
- grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l
- grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'
- python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20

Accept when:
- All modules implementing send() methods or message queue boundaries contain logging.getLogger(__name__) initialization
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules

## Enforcement

- Verified by: Code review checklist requiring logger initialization in modules with transmission boundaries
- Verified by: Static analysis or linting rules to detect send() methods without corresponding logger initialization
- Verified by: CI pipeline checks for logging patterns in modified files that implement I/O operations
- Violation handling: Code review feedback requesting addition of logging infrastructure before merge
- Violation handling: CI warnings (non-blocking initially) for missing loggers in transmission boundary modules
- Violation handling: Periodic audits of modules implementing send() methods to ensure compliance
- Exception process: Document rationale for exception in module docstring or ADR
- Exception process: Obtain approval from tech lead or architect for performance-critical paths where logging overhead is prohibitive
- Exception process: Implement alternative observability mechanism (metrics, tracing) if logging is exempted
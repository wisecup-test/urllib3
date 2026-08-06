# Standardize Socket Lifecycle Management Through Custom Config Classes: Dual Stack Socket

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires precise control over socket creation, binding, and lifecycle management across multiple server implementations (Hypercorn, ASGI proxy)
- Test infrastructure demands dual-stack IPv4/IPv6 socket binding to avoid timeout delays on Windows where urllib3 attempts IPv6 first without Happy Eyeballs
- Server configuration classes need to override framework defaults to implement retry logic for EADDRINUSE errors in crowded CI environments
- Public API contracts expose socket lifecycle methods (create_sockets, bind, close, readable, writable) and connection management (connect, start_forward) as extension points

## Problem Statement

Server implementations using standard framework socket creation patterns experience IPv6 binding failures and race conditions in CI environments, leading to test timeouts and flaky builds. The default socket creation behavior does not guarantee dual-stack binding or provide retry mechanisms for transient address-in-use errors.

## Decision

1. MUST: Dual-stack socket creation MUST bind both IPv4 and IPv6 addresses when host is localhost to prevent Happy Eyeballs timeout delays

## Policy Block

- MUST Dual-stack socket creation MUST bind both IPv4 and IPv6 addresses when host is localhost to prevent Happy Eyeballs timeout delays

In scope:
- Server configuration classes extending framework config (hypercorn.Config, ASGI applications)
- Socket creation and binding logic in test infrastructure (dummyserver)
- Proxy implementations requiring connection forwarding (asgi_proxy)
- Browser-based fetch implementations using abort controllers (emscripten/fetch)

Out of scope:
- Client-side socket creation (httpx.AsyncClient, urllib3 connection pools)
- Production server deployments using standard framework socket creation
- Single-stack IPv4-only or IPv6-only environments
- Non-socket transport mechanisms (Unix domain sockets, named pipes)

Exceptions:
- EX-001: Production environments with dedicated IP addresses where EADDRINUSE is not expected

## Rationale

- Evidence shows 3 files implementing custom socket lifecycle management with retry logic, dual-stack binding, and public API contracts for extension
- The pattern addresses concrete CI failures (EADDRINUSE) and Windows test timeouts (IPv6 Happy Eyeballs delays) through systematic retry and dual-stack binding
- Public API contracts (create_sockets, bind, connect, start_forward, close, readable, writable) enable framework extension without forking server implementations
- Socket option configuration (TCP_NODELAY, SO_REUSEADDR, setblocking) follows established patterns for high-performance server implementations

## Consequences

Positive:
- Eliminates IPv6 timeout delays on Windows by ensuring dual-stack binding for localhost
- Reduces CI flakiness through systematic retry of EADDRINUSE errors (up to 10 attempts)
- Provides clear extension points for custom socket lifecycle management without framework modifications
- Enables consistent socket configuration (TCP_NODELAY, SO_REUSEADDR) across server implementations

Negative:
- Increases complexity of server configuration classes with retry logic and dual-stack handling
- Requires maintenance of framework-specific override methods (create_sockets) across version upgrades
- Retry logic may mask underlying network configuration issues in development environments
- Public API contracts create backward compatibility obligations for socket lifecycle methods

## Alternatives

- Use framework default socket creation without custom overrides (rejected)
  Rejected because: Framework defaults do not provide dual-stack binding for localhost or retry logic for EADDRINUSE, causing test timeouts and CI failures
  When valid: Production environments with dedicated IPs where dual-stack binding and retry logic are not required
- Implement socket retry logic at the test harness level rather than in configuration classes (rejected)
  Rejected because: Test harness retry would not address the root cause of IPv6 timeout delays and would require duplicating retry logic across multiple test suites
  When valid: When socket binding failures are truly transient and not systematic across CI environments
- Use separate IPv4 and IPv6 ports instead of dual-stack binding to the same port (rejected)
  Rejected because: Separate ports would require test clients to attempt both ports, increasing test complexity and not solving the Happy Eyeballs timeout issue
  When valid: When clients explicitly target specific protocol versions rather than relying on getaddrinfo resolution

## Risks

- Framework API changes in Hypercorn or other servers may break custom create_sockets overrides
  Mitigation: Pin framework versions in requirements and add integration tests that verify socket creation behavior across upgrades
  Owner: Infrastructure team
- Retry logic may hide legitimate network configuration errors in development environments
  Mitigation: Log all retry attempts with errno details to stderr and fail after 10 attempts with clear error messages
  Owner: Engineering team
- Public API contracts for socket lifecycle methods create backward compatibility obligations
  Mitigation: Document socket lifecycle methods as stable APIs with semantic versioning guarantees and deprecation policies
  Owner: API design team

## Implementation Notes

- Override framework config classes (e.g., hypercorn.Config) and implement create_sockets to return dual-stack socket lists
- Use socket.getaddrinfo with AF_UNSPEC or AF_INET6 to resolve both IPv4 and IPv6 addresses for localhost
- Wrap socket creation in retry loop (10 attempts) catching OSError with errno.EADDRINUSE, logging each retry to stderr
- Set socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) before binding
- Expose socket lifecycle methods (create_sockets, bind, close, readable, writable) and connection methods (connect, start_forward) as public API contracts with docstrings

## Continuation Context


Verify commands:
- grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'
- grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'
- grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'
- grep -r 'TCP_NODELAY\|SO_REUSEADDR' --include='*.py' | grep -v '__pycache__'

Accept when:
- All server configuration classes override framework socket creation methods and implement retry logic for EADDRINUSE
- Socket creation for localhost binds both IPv4 and IPv6 addresses to the same port
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently set before binding across all implementations
- Public API contracts for socket lifecycle methods are documented and exposed in server configuration classes

## Enforcement

- Verified by: CI integration tests verify dual-stack socket binding on Windows and Linux
- Verified by: Code review checklist requires socket lifecycle override verification for new server implementations
- Verified by: Automated grep patterns in CI check for EADDRINUSE retry logic and socket option configuration
- Violation handling: CI fails if socket creation does not implement retry logic or dual-stack binding
- Violation handling: Code review blocks merge if new server configurations do not override create_sockets with documented rationale
- Violation handling: Test failures on Windows due to IPv6 timeouts trigger automatic review of socket binding implementation
- Exception process: Request exception through infrastructure team with documented justification for single-stack or no-retry configuration
- Exception process: Provide evidence that target environment does not require dual-stack binding or retry logic
- Exception process: Document exception in server configuration class docstring with environment-specific constraints
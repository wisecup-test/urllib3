# Standardize Socket Binding via sock.bind() for Network Server Initialization: Socket Binding Failures

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements multiple network server components (hypercornserver.py, socketserver.py) that require low-level socket initialization and binding to network interfaces
- Socket binding operations appear in server configuration classes that integrate with frameworks like Hypercorn, requiring custom socket creation to control IPv4/IPv6 dual-stack behavior and port allocation
- The pattern emerges from the need to create sockets with specific options (TCP_NODELAY, SO_REUSEADDR) and handle binding failures gracefully in CI environments where ports may be contested
- Browser-based networking via Emscripten (fetch.py) uses JavaScript abort controllers bound to request lifecycle, representing an alternative binding pattern for non-socket network primitives

## Problem Statement

Network server initialization requires consistent, reliable socket binding that handles dual-stack IPv4/IPv6 requirements, port allocation conflicts in concurrent test environments, and integration with async server frameworks while maintaining control over socket options and error handling strategies.

## Decision

1. MUST: Socket binding failures with errno.EADDRINUSE MUST implement retry logic with a minimum of 10 attempts before raising OSError

## Policy Block

- MUST Socket binding failures with errno.EADDRINUSE MUST implement retry logic with a minimum of 10 attempts before raising OSError

In scope:
- All TCP server socket initialization in dummyserver components
- Custom socket creation methods in Hypercorn configuration classes
- Socket binding operations in test server infrastructure
- Network primitive lifecycle management in Emscripten/JavaScript bridge code

Out of scope:
- Client-side socket connections (connect operations)
- UDP socket binding patterns
- Unix domain socket creation
- High-level HTTP client request APIs that abstract socket operations

## Rationale

- The pattern is observed across 3 files with 91.47% confidence, demonstrating consistent socket binding practices in server initialization code paths
- Dual-stack IPv4/IPv6 support is critical for test infrastructure that must work across diverse CI environments and avoid 2-second timeouts from IPv6 connection attempts
- Retry logic for EADDRINUSE errors addresses real-world port contention in concurrent test execution environments, as evidenced by explicit retry implementation with stderr logging
- The sock.bind() pattern provides explicit control over network interface binding, port allocation, and socket options that higher-level abstractions do not expose

## Consequences

Positive:
- Consistent socket initialization across server components reduces debugging complexity and improves maintainability
- Explicit dual-stack handling eliminates IPv6-related test timeouts and improves test suite performance
- Retry logic for port binding increases reliability in contested CI environments with parallel test execution
- Low-level socket control enables optimization of TCP options (TCP_NODELAY, SO_REUSEADDR) for test server performance

Negative:
- Direct socket manipulation increases code complexity compared to using framework-provided socket creation
- Retry logic with fixed iteration count (10 attempts) may still fail in extremely contested environments
- Platform-specific socket behavior (IPv6 availability, port allocation) requires careful testing across operating systems
- Tight coupling to socket module APIs reduces portability to non-CPython environments without socket support

## Alternatives

- Use Hypercorn's default socket creation without custom override (rejected)
  Rejected because: Default Hypercorn socket creation binds only to IPv4 when using localhost:0, causing 2-second timeouts on Windows when urllib3 attempts IPv6 first without Happy Eyeballs implementation
  When valid: When test infrastructure runs only on IPv4-only networks or when Happy Eyeballs is implemented in the HTTP client
- Implement socket binding at the application layer rather than in server configuration (rejected)
  Rejected because: Server frameworks like Hypercorn require sockets to be created and bound before server startup, necessitating configuration-time socket creation
  When valid: When using server frameworks that accept pre-bound sockets as runtime parameters rather than configuration-time initialization
- Use higher-level networking libraries that abstract socket binding (deferred)
  When valid: For production server code where test-specific dual-stack requirements and port contention handling are not primary concerns

## Risks

- Socket binding retry logic may introduce test flakiness if port contention exceeds 10 retry attempts in heavily loaded CI environments
  Mitigation: Monitor CI logs for EADDRINUSE retry messages and increase retry count if failures occur; consider exponential backoff between retries
  Owner: Test Infrastructure Team
- Platform-specific socket behavior differences (Windows vs Linux vs macOS) may cause inconsistent test results or binding failures
  Mitigation: Maintain comprehensive test coverage across all supported platforms; document platform-specific socket behavior in code comments
  Owner: Engineering Team
- Direct socket manipulation bypasses framework safety checks and error handling, potentially exposing edge cases in socket lifecycle management
  Mitigation: Implement comprehensive error handling for all socket operations; ensure proper socket cleanup in exception paths using context managers
  Owner: Engineering Team

## Implementation Notes

- When implementing custom socket creation, always use socket.getaddrinfo() to resolve host/port combinations and iterate over all returned address families to support dual-stack binding
- Set socket options (TCP_NODELAY, SO_REUSEADDR) immediately after socket creation and before binding to ensure they take effect
- When binding to port 0 for automatic allocation, extract the assigned port using sock.getsockname()[1] and reuse it for subsequent socket bindings to maintain consistent port across IPv4/IPv6
- Wrap socket binding operations in try-except blocks that specifically catch OSError with errno.EADDRINUSE, and log retry attempts to stderr for debugging CI failures

## Continuation Context


Verify commands:
- grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test
- grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'
- grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'

Accept when:
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling

## Enforcement

- Verified by: Code review checklist requiring verification of socket binding patterns in server initialization code
- Verified by: Automated grep-based verification in CI pipeline checking for sock.bind() usage and EADDRINUSE handling
- Verified by: Cross-platform test suite execution validating dual-stack socket binding on Linux, Windows, and macOS
- Violation handling: Code review rejection for server socket initialization that does not follow sock.bind() pattern
- Violation handling: CI pipeline warnings for missing EADDRINUSE retry logic in socket binding code
- Violation handling: Test failures on platforms where dual-stack binding is not properly implemented
- Exception process: Document exception rationale in code comments explaining why alternative socket binding approach is required
- Exception process: Obtain approval from test infrastructure team for deviations from standard socket binding patterns
- Exception process: Add platform-specific conditional logic with clear comments when socket behavior must differ across operating systems
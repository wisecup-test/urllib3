# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Test Servers Binding

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test server infrastructure requires socket binding operations that support both IPv4 and IPv6 to avoid connection delays in test environments, particularly on Windows where IPv6 is attempted first without Happy Eyeballs implementation
- Multiple server implementations (hypercornserver.py, socketserver.py) independently implement socket binding with host and port configuration using socket.bind((host, port)) patterns
- The emscripten fetch module implements abort controller binding (js_abort_controller.abort.bind) for JavaScript interop in WebAssembly environments, indicating cross-platform API surface requirements
- Socket creation involves retry logic to handle EADDRINUSE errors in crowded CI environments, with up to 10 retry attempts to ensure reliable port allocation across dual-stack configurations

## Problem Statement

Test server infrastructure must reliably bind sockets across IPv4 and IPv6 address families while handling port allocation conflicts in CI environments, but inconsistent socket binding implementations create maintenance burden and risk connection timeout failures when clients attempt IPv6 connections to IPv4-only bound servers.

## Decision

1. MUST: Test servers binding to localhost MUST support dual-stack IPv4 and IPv6 by iterating through socket.getaddrinfo() results with socket.AF_UNSPEC or socket.AF_INET6 family

## Policy Block

- MUST Test servers binding to localhost MUST support dual-stack IPv4 and IPv6 by iterating through socket.getaddrinfo() results with socket.AF_UNSPEC or socket.AF_INET6 family

In scope:
- Test server socket binding in dummyserver/hypercornserver.py
- Test server socket binding in dummyserver/socketserver.py
- Socket creation for HTTP/HTTPS test endpoints
- Dual-stack IPv4/IPv6 localhost binding operations
- Port allocation with retry logic in CI environments

Out of scope:
- Production server socket binding configurations
- Client-side socket connection logic
- Non-test infrastructure socket operations
- UDP or QUIC socket binding patterns
- Operating system-level network configuration

Exceptions:
- EXC-001: Platform does not support IPv6 (detected via socket.getaddrinfo returning only IPv4 results)
- EXC-002: WebAssembly/Emscripten environments require JavaScript function binding for abort controllers or fetch APIs

## Rationale

- Evidence shows 3 files implementing socket binding patterns with sock.bind((host, port)) across test server infrastructure, indicating a consistent architectural approach to network endpoint creation
- The hypercornserver.py implementation explicitly documents IPv6 binding requirements to avoid 2-second timeouts on Windows when urllib3 attempts IPv6 first, demonstrating performance impact of IPv4-only binding
- Retry logic with errno.EADDRINUSE handling addresses real-world CI environment constraints where port conflicts occur frequently, with 10 retry attempts providing sufficient resilience
- Dual-stack binding with port reuse across address families ensures consistent endpoint addressing for test clients while maintaining compatibility with both IPv4 and IPv6 network stacks

## Consequences

Positive:
- Eliminates 2-second connection timeout delays on Windows test runs by ensuring IPv6 sockets are bound when clients attempt IPv6 connections first
- Reduces CI flakiness from EADDRINUSE errors through systematic retry logic across all test server implementations
- Provides consistent socket binding patterns across multiple server implementations (Hypercorn, custom socketserver), reducing maintenance burden
- Enables test infrastructure to work correctly in dual-stack network environments without requiring IPv4-only or IPv6-only configuration

Negative:
- Increases complexity of socket binding code with retry logic and dual-stack iteration through getaddrinfo results
- Retry logic with 10 attempts may add latency to test startup in pathological cases where ports are consistently unavailable
- Dual-stack binding requires careful port number coordination to ensure IPv4 and IPv6 sockets use the same port, adding implementation complexity
- Platform-specific behavior differences in socket.getaddrinfo may require additional testing and edge case handling

## Alternatives

- Bind only to IPv4 sockets and rely on IPv4-mapped IPv6 addresses for dual-stack support (rejected)
  Rejected because: Windows environments do not reliably support IPv4-mapped IPv6 addresses, leading to connection timeouts when clients attempt IPv6 first without Happy Eyeballs
  When valid: On Linux systems with proper IPv4-mapped IPv6 kernel support where test performance is not critical
- Use operating system-level socket activation or systemd socket units for test servers (rejected)
  Rejected because: Test infrastructure requires programmatic control over socket creation and port allocation, and systemd socket activation is not available on Windows or macOS CI environments
  When valid: For production deployments on Linux systems with systemd where socket lifecycle is managed externally
- Implement single-attempt socket binding without retry logic and fail fast on EADDRINUSE (rejected)
  Rejected because: CI environments with parallel test execution frequently encounter port conflicts, and failing fast would significantly increase test flakiness and CI failure rates
  When valid: In isolated test environments with guaranteed port availability and no concurrent test execution

## Risks

- Retry logic may mask underlying port exhaustion issues in CI environments, delaying detection of resource constraints
  Mitigation: Log each retry attempt with stderr output to provide visibility into binding failures and retry frequency; monitor CI logs for excessive retry patterns
  Owner: Test Infrastructure Team
- Platform-specific differences in socket.getaddrinfo behavior may cause inconsistent dual-stack binding across operating systems
  Mitigation: Implement comprehensive test coverage for socket binding on Windows, Linux, and macOS; document platform-specific behavior in code comments
  Owner: Engineering Team
- Port reuse logic may fail if IPv4 and IPv6 port namespaces are not unified on certain platforms or network configurations
  Mitigation: Validate port consistency after binding by checking sock.getsockname()[1] for all created sockets; fall back to separate ports if unified port allocation fails
  Owner: Engineering Team

## Implementation Notes

- Use socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM, 0, socket.AI_PASSIVE) to retrieve all available address families for the specified host
- After binding the first socket with port 0, extract the allocated port using sock.getsockname()[1] and reuse this port for subsequent address family bindings
- Wrap socket binding in a retry loop that catches OSError with errno.EADDRINUSE, logs the retry attempt to stderr, and attempts up to 10 times before raising the exception
- Configure sockets with sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1) and sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) before binding
- Set sock.setblocking(False) and sock.set_inheritable(True) after successful binding to ensure non-blocking operation and proper subprocess inheritance

## Continuation Context


Verify commands:
- grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'
- grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'
- grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'

Accept when:
- All test server implementations in dummyserver/ use socket.bind((host, port)) with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through socket.getaddrinfo results with AF_UNSPEC or AF_INET6
- Retry logic with errno.EADDRINUSE handling is present in socket binding code with at least 10 retry attempts

## Enforcement

- Verified by: Code review of all test server socket binding implementations
- Verified by: Automated grep-based verification in CI pipeline checking for required socket.bind patterns
- Verified by: Integration tests validating dual-stack connectivity on Windows, Linux, and macOS environments
- Violation handling: CI pipeline fails if grep verification commands do not find required socket binding patterns
- Violation handling: Code review blocks merge if new test server implementations do not follow dual-stack binding requirements
- Violation handling: Test failures on Windows indicating IPv6 connection timeouts trigger investigation of socket binding implementation
- Exception process: Document platform-specific limitations in code comments if dual-stack binding is not feasible
- Exception process: Obtain architecture team approval for alternative socket binding approaches with justification
- Exception process: Update ADR with approved exceptions and their specific applicability conditions
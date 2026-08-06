# Standardize Socket Binding and Core Library Usage for Network Server Components: Core Library Imports

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Network server components in dummyserver/hypercornserver.py and dummyserver/socketserver.py implement custom socket binding logic using Python's socket module with explicit host and port configuration
- The hypercornserver.py module extends hypercorn.Config to override create_sockets() and implements retry logic for socket binding with dual IPv4/IPv6 support to avoid test failures on Windows
- Socket configuration includes TCP_NODELAY, SO_REUSEADDR options, and non-blocking mode with inheritable file descriptors for process management
- The emscripten fetch module uses JavaScript interop (js module) for browser-based networking with abort controllers bound to JavaScript objects
- Core libraries detected across all three files include socket, sys, contextlib, errno, functools, logging, os, ssl, io, json, and email.parser for foundational networking and I/O operations

## Problem Statement

Network server components require consistent socket binding patterns and core library usage to ensure reliable dual-stack IPv4/IPv6 support, proper resource management, and cross-platform compatibility, particularly in test environments where port conflicts and address family mismatches can cause intermittent failures.

## Decision

1. MUST: Core library imports MUST include socket, sys, and contextlib for foundational networking and resource management

## Policy Block

- MUST Core library imports MUST include socket, sys, and contextlib for foundational networking and resource management

In scope:
- Network server components in dummyserver modules
- Socket binding and configuration logic
- Test server infrastructure requiring dual-stack IPv4/IPv6 support
- Hypercorn configuration extensions
- Browser-based fetch implementations using Emscripten

Out of scope:
- Client-side HTTP connection logic
- Production server deployment configurations
- Third-party server framework internals
- Operating system socket implementation details

## Rationale

- The pattern emerges from 3 files with 91.47% confidence, demonstrating consistent socket binding practices across test server infrastructure
- Retry logic for socket binding addresses real-world port conflicts in CI environments, particularly on Windows where IPv6 binding failures waste approximately 2 seconds per test
- Dual-stack IPv4/IPv6 support prevents Happy Eyeballs timeout delays by ensuring both address families are available on localhost
- Core library usage (socket, sys, contextlib, errno, functools, logging, ssl, io, json) establishes a stable foundation for network operations with proper error handling and resource management

## Consequences

Positive:
- Consistent socket binding patterns reduce intermittent test failures caused by port conflicts and address family mismatches
- Retry logic with EADDRINUSE handling improves reliability in crowded CI environments
- Dual-stack IPv4/IPv6 support eliminates 2-second delays per test on Windows by avoiding Happy Eyeballs timeouts
- Standardized core library usage ensures proper resource management and error handling across network components

Negative:
- Retry logic adds complexity to socket creation code and may mask underlying port allocation issues
- Dual-stack binding requires additional socket creation and management overhead
- Platform-specific behavior (Windows IPv6 handling) increases testing surface area
- Custom Hypercorn configuration overrides may diverge from upstream framework patterns

## Alternatives

- Use operating system default socket binding without retry logic (rejected)
  Rejected because: Causes intermittent EADDRINUSE failures in CI environments with concurrent test execution, particularly on crowded runners
  When valid: Single-threaded test execution with guaranteed port availability
- Bind only to IPv4 and rely on IPv4-mapped IPv6 addresses (rejected)
  Rejected because: Wastes approximately 2 seconds per test on Windows as urllib3 tries IPv6 first without Happy Eyeballs implementation
  When valid: IPv6 is disabled or not required for test coverage
- Use Hypercorn's default socket creation without custom overrides (rejected)
  Rejected because: Hypercorn only binds to IPv4 when requesting localhost with port zero, missing IPv6 support needed for test reliability
  When valid: IPv6 test coverage is not required or tests run on IPv4-only systems

## Risks

- Retry logic may hide systematic port allocation problems or resource leaks in test infrastructure
  Mitigation: Log retry attempts to stderr and fail after 10 attempts to surface persistent issues
  Owner: engineering team
- Platform-specific socket behavior differences between Windows, Linux, and macOS may cause inconsistent test results
  Mitigation: Maintain cross-platform CI coverage and document platform-specific socket configuration requirements
  Owner: engineering team
- Custom Hypercorn configuration overrides may break compatibility with future Hypercorn versions
  Mitigation: Pin Hypercorn version dependencies and test configuration overrides during upgrades
  Owner: engineering team

## Implementation Notes

- Implement socket binding retry logic with exponential backoff up to 10 attempts, logging each EADDRINUSE error to stderr for debugging
- Use socket.getaddrinfo with AF_UNSPEC to discover both IPv4 and IPv6 addresses, binding to the same port number for both families
- Configure sockets with TCP_NODELAY and SO_REUSEADDR immediately after creation and before binding
- Set sock.set_inheritable(True) for sockets that need to be passed to child processes in server implementations
- For browser-based networking in Emscripten environments, bind JavaScript abort controllers using .bind() to maintain proper this context

## Continuation Context


Verify commands:
- grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/
- grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/
- grep -r 'errno\.EADDRINUSE' dummyserver/
- python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'

Accept when:
- Socket binding operations use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules

## Enforcement

- Verified by: Automated grep patterns in CI checking for sock.bind() usage and socket option configuration
- Verified by: Code review verification of retry logic and dual-stack IPv4/IPv6 support in server components
- Verified by: Cross-platform test execution on Windows, Linux, and macOS to validate socket binding behavior
- Violation handling: CI pipeline fails if socket binding patterns do not include required TCP_NODELAY and SO_REUSEADDR options
- Violation handling: Code review blocks merge if retry logic is missing from new socket binding implementations
- Violation handling: Test failures on Windows due to IPv6 timeout delays trigger investigation of dual-stack binding compliance
- Exception process: Document platform-specific socket requirements that prevent standard binding patterns
- Exception process: Obtain approval from maintainers for alternative socket configuration approaches
- Exception process: Add inline comments explaining why standard patterns cannot be applied
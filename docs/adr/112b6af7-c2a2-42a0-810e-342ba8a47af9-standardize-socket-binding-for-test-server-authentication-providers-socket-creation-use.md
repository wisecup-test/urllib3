# Standardize Socket Binding for Test Server Authentication Providers: Socket Creation Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test server infrastructure requires dynamic socket binding to support concurrent test execution without port conflicts
- Multiple server implementations (Hypercorn, socketserver) use direct socket.bind() calls with host and port parameters to establish network listeners
- IPv6 and IPv4 dual-stack binding is necessary to avoid test delays on systems where urllib3 attempts IPv6 first
- Port allocation failures (EADDRINUSE) occur frequently in CI environments, requiring retry logic for reliable test execution
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options to optimize connection handling and enable rapid socket reuse

## Problem Statement

Test server implementations need a consistent approach to socket binding that handles dual-stack networking, port conflicts in concurrent environments, and socket option configuration while maintaining compatibility with both secure and insecure connection modes across different server frameworks.

## Decision

1. MUST: Socket creation MUST use socket.getaddrinfo() to resolve host addresses and support both IPv4 and IPv6

## Policy Block

- MUST Socket creation MUST use socket.getaddrinfo() to resolve host addresses and support both IPv4 and IPv6

In scope:
- Test server socket creation in dummyserver modules
- Hypercorn configuration socket binding
- Socket-based authentication provider setup
- IPv4 and IPv6 dual-stack socket allocation

Out of scope:
- Production server socket configuration
- Client-side socket connections
- Unix domain sockets
- QUIC socket configuration

## Rationale

- Evidence shows consistent use of sock.bind((host, port)) pattern across dummyserver/hypercornserver.py and dummyserver/socketserver.py with 91.47% confidence
- The pattern addresses real operational challenges: IPv6 binding failures waste ~2 seconds per test on Windows, and EADDRINUSE errors cause test failures in CI
- Socket option configuration (TCP_NODELAY, SO_REUSEADDR) appears consistently across implementations, indicating established best practices
- Retry logic and dual-stack support demonstrate mature handling of concurrent test execution requirements

## Consequences

Positive:
- Consistent socket binding approach across test server implementations reduces maintenance burden
- Dual-stack IPv4/IPv6 support eliminates test delays from sequential connection attempts
- Retry logic significantly improves test reliability in concurrent CI environments
- Port 0 allocation enables safe parallel test execution without manual port management

Negative:
- Retry logic adds complexity and potential latency (up to 10 attempts) when port conflicts occur
- Dual-stack binding requires careful port coordination to avoid IPv4/IPv6 port mismatches
- Non-blocking socket mode requires compatible server frameworks and event loop integration
- Socket option configuration may not be portable across all operating systems

## Alternatives

- Use high-level server framework socket management (e.g., Hypercorn's default socket creation) (rejected)
  Rejected because: Framework defaults do not handle dual-stack binding requirements, causing IPv6 connection delays and test failures on Windows
  When valid: When IPv6 support is not required and tests run sequentially without port conflicts
- Pre-allocate fixed port ranges for test servers (rejected)
  Rejected because: Fixed ports prevent parallel test execution and create conflicts in shared CI environments
  When valid: In isolated test environments with guaranteed exclusive port access
- Use operating system socket activation (systemd socket activation) (rejected)
  Rejected because: Adds external dependencies and complexity inappropriate for lightweight test infrastructure
  When valid: For production deployments requiring zero-downtime restarts

## Risks

- Retry logic may fail to bind after 10 attempts in extremely congested CI environments
  Mitigation: Monitor binding failure rates and increase retry count if needed; implement exponential backoff between attempts
  Owner: engineering team
- IPv6 binding may fail on systems with IPv6 disabled or misconfigured
  Mitigation: Gracefully handle IPv6 binding failures and fall back to IPv4-only mode; log warnings for debugging
  Owner: engineering team
- Socket options (TCP_NODELAY, SO_REUSEADDR) may behave differently across operating systems
  Mitigation: Test socket behavior on all supported platforms (Linux, Windows, macOS); document platform-specific behaviors
  Owner: engineering team

## Implementation Notes

- Use socket.getaddrinfo() with socket.AI_PASSIVE flag to resolve bind addresses for both IPv4 and IPv6
- Extract port number from first bound socket using sock.getsockname()[1] and reuse for subsequent dual-stack bindings
- Wrap socket binding in try-except blocks catching OSError with errno.EADDRINUSE for retry logic
- Set socket.set_inheritable(True) to allow socket passing to child processes if needed
- Log retry attempts to stderr for debugging CI failures: print(f'Retrying binding to {bind} after EADDRINUSE', file=sys.stderr)

## Continuation Context


Verify commands:
- grep -r 'sock\.bind((.*host.*port.*))' dummyserver/
- grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'
- grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l

Accept when:
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count

## Enforcement

- Verified by: Code review of test server implementations
- Verified by: Automated grep patterns in CI checking for socket.bind() usage
- Verified by: Integration tests validating dual-stack binding and port allocation
- Violation handling: CI pipeline fails if socket binding patterns do not match required structure
- Violation handling: Code review blocks merge if retry logic is missing from new server implementations
- Violation handling: Test failures on Windows indicate IPv6 binding issues requiring remediation
- Exception process: Document platform-specific socket limitations in code comments
- Exception process: Request architecture review for alternative binding approaches
- Exception process: Obtain approval from test infrastructure maintainers for deviations
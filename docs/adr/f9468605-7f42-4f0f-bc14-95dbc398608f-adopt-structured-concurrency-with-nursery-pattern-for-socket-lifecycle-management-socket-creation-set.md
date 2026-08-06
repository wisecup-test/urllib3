# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Socket Creation Set

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements socket-based authentication server infrastructure using hypercorn.trio.serve with structured concurrency primitives
- Socket creation and binding operations require retry logic to handle port conflicts in IPv4/IPv6 dual-stack environments, particularly in CI environments
- Asynchronous request handling uses nursery.start() pattern to coordinate server lifecycle with shutdown events
- Emscripten fetch operations use JavaScript abort controllers bound to request lifecycle for cancellation semantics
- The pattern emerges from the need to manage concurrent socket operations, HTTP server lifecycle, and request cancellation in a structured manner

## Problem Statement

Authentication server implementations require coordinated management of socket lifecycle, concurrent request handling, and graceful shutdown across multiple protocol families (IPv4/IPv6) and runtime environments (native Python, Emscripten). Unstructured concurrency approaches lead to resource leaks, race conditions in port binding, and incomplete cleanup during server shutdown.

## Decision

1. MUST: Socket creation MUST set TCP_NODELAY and SO_REUSEADDR options to optimize latency and enable rapid server restart

## Policy Block

- MUST Socket creation MUST set TCP_NODELAY and SO_REUSEADDR options to optimize latency and enable rapid server restart

In scope:
- Socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers
- Dual-stack IPv4/IPv6 socket binding operations
- Structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.)
- Request cancellation in browser environments via Emscripten/JavaScript interop

Out of scope:
- Synchronous blocking socket servers without concurrency requirements
- Single address family (IPv4-only or IPv6-only) deployments
- Authentication mechanisms not involving socket lifecycle management
- Thread-based concurrency models without structured cancellation

Exceptions:
- EXC-001: Platform does not support dual-stack sockets (e.g., IPv6 disabled at OS level)
- EXC-002: Legacy codebase migration where structured concurrency refactor exceeds available resources

## Rationale

- The evidence shows explicit use of nursery.start with functools.partial to coordinate hypercorn.trio.serve lifecycle with shutdown_event.wait, demonstrating structured concurrency for resource cleanup
- Retry logic with errno.EADDRINUSE handling (up to 10 attempts) addresses real-world port conflicts in CI environments where IPv4/IPv6 port allocation races occur
- Dual-stack socket creation using socket.getaddrinfo with AF_UNSPEC ensures both IPv4 and IPv6 binding, eliminating 2-second connection delays observed on Windows when urllib3 attempts IPv6 first
- JavaScript abort controller binding (js_abort_controller.abort.bind) in Emscripten context provides cancellation semantics consistent with structured concurrency principles across runtime boundaries

## Consequences

Positive:
- Guaranteed cleanup of socket resources through structured concurrency scope management, preventing file descriptor leaks
- Elimination of IPv6 connection timeout delays (2+ seconds per request) in dual-stack environments
- Graceful shutdown coordination between server lifecycle and active request handling
- Consistent cancellation semantics across native Python and Emscripten/JavaScript runtime boundaries

Negative:
- Increased complexity in socket creation logic due to retry mechanisms and dual-stack handling
- Dependency on structured concurrency framework (Trio) limits portability to environments without such primitives
- Retry logic with 10 attempts may delay server startup by several seconds in pathological port conflict scenarios
- Abort controller binding in Emscripten adds JavaScript interop overhead for each cancellable request

## Alternatives

- Use thread-based concurrency with manual socket cleanup in finally blocks (rejected)
  Rejected because: Manual cleanup is error-prone and does not provide cancellation propagation; evidence shows structured concurrency with nursery.start provides automatic resource cleanup
  When valid: Legacy systems where Trio/asyncio migration is not feasible and resource leaks are acceptable
- Bind only to IPv4 and accept IPv6 connection delays (rejected)
  Rejected because: Evidence documents 2-second delays per test on Windows; dual-stack binding eliminates this performance penalty
  When valid: IPv4-only networks or deployments where IPv6 is explicitly disabled
- Use OS-level socket activation (systemd socket activation) instead of application-level binding (deferred)
  Rejected because: Not rejected; could complement current approach but requires deployment infrastructure changes
  When valid: Production deployments with systemd where socket activation provides additional benefits like zero-downtime restart

## Risks

- Retry logic may fail to bind sockets after 10 attempts in extremely congested CI environments, causing test failures
  Mitigation: Monitor EADDRINUSE failure rates in CI; increase retry count or implement exponential backoff if failures exceed 1%
  Owner: Infrastructure team
- Structured concurrency framework (Trio) may have compatibility issues with future Python async/await evolution
  Mitigation: Abstract concurrency primitives behind interface layer; monitor PEP proposals for asyncio TaskGroup standardization
  Owner: Engineering team
- JavaScript abort controller binding may fail in non-browser Emscripten environments without DOM APIs
  Mitigation: Feature-detect abort controller availability; fall back to timeout-based cancellation if unavailable
  Owner: Web platform team

## Implementation Notes

- Wrap hypercorn.trio.serve calls with nursery.start(functools.partial(..., shutdown_trigger=event.wait)) to ensure shutdown coordination
- Implement socket creation with socket.getaddrinfo(host, port, AF_UNSPEC, SOCK_STREAM, 0, AI_PASSIVE) to enable dual-stack binding
- Set socket options TCP_NODELAY and SO_REUSEADDR immediately after socket creation and before bind()
- Wrap socket binding in retry loop catching OSError with errno.EADDRINUSE; log retry attempts to stderr for debugging
- In Emscripten contexts, create abort controller and bind abort method before initiating fetch operations for cancellation support

## Continuation Context


Verify commands:
- grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .
- grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .
- grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'

Accept when:
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost
- Cancellable request operations in Emscripten context use abort controller binding pattern

## Enforcement

- Verified by: Code review checklist verifying nursery.start usage in server startup code
- Verified by: Automated grep-based CI check for AF_UNSPEC and TCP_NODELAY in socket creation
- Verified by: Integration tests validating dual-stack binding and graceful shutdown behavior
- Violation handling: PR comments requesting refactor to structured concurrency pattern with example code
- Violation handling: CI failure on missing dual-stack socket configuration in new server implementations
- Violation handling: Architecture review escalation for violations in authentication-critical code paths
- Exception process: Document exception rationale in ADR exceptions section with approval from technical lead
- Exception process: Add inline code comments explaining why structured concurrency is not applicable
- Exception process: Include mitigation plan for resource cleanup in exception approval
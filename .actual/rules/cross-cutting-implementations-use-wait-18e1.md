# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Implementations Use Wait

These rules are ALWAYS ACTIVE for all socket-based HTTP/HTTPS authentication server implementations using hypercorn or similar ASGI servers with structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.), dual-stack IPv4/IPv6 socket binding operations, and request cancellation in browser environments via Emscripten/JavaScript interop.

### Rules

- **R-CONC-001** MAY: Implementations MAY use wait_for_streaming_ready patterns to coordinate streaming request initialization with server readiness.
- **R-CONC-002** MUST: Socket-based server implementations MUST use nursery.start or equivalent structured concurrency primitive for lifecycle management and graceful shutdown coordination.
- **R-CONC-003** MUST: Socket binding code MUST include retry logic handling EADDRINUSE (errno.EADDRINUSE) with up to 10 retry attempts to address port conflicts in CI environments.
- **R-CONC-004** MUST: Socket creation MUST use socket.getaddrinfo with AF_UNSPEC to enable dual-stack IPv4/IPv6 binding when host is localhost or 0.0.0.0.
- **R-CONC-005** MUST: Socket options TCP_NODELAY and SO_REUSEADDR MUST be set immediately after socket creation and before bind() operation.
- **R-CONC-006** SHOULD: Cancellable request operations in Emscripten context SHOULD use abort controller binding pattern (js_abort_controller.abort.bind) for cancellation support.
- **R-CONC-007** SHOULD: Retry attempts for socket binding SHOULD be logged to stderr for debugging and monitoring purposes.

### Verify

```bash
# Verify nursery.start usage with functools.partial for hypercorn lifecycle management
grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .

# Verify dual-stack socket configuration with AF_UNSPEC and TCP_NODELAY
grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .

# Verify EADDRINUSE retry logic is present
grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'

# Verify SO_REUSEADDR is set
grep -r 'SO_REUSEADDR' --include='*.py' .

# Verify shutdown event coordination
grep -r 'shutdown.*event\.wait\|shutdown_trigger' --include='*.py' .
```

**Accept when:**
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE with loop/range construct and binds to both IPv4 and IPv6 when host is localhost or 0.0.0.0
- Socket creation uses socket.getaddrinfo with AF_UNSPEC parameter
- TCP_NODELAY and SO_REUSEADDR socket options are set before bind() operation
- Graceful shutdown is coordinated through structured concurrency scope (nursery) with shutdown event wait pattern
- Cancellable request operations in Emscripten context use abort controller binding pattern
- Retry attempts are logged for debugging and monitoring

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket-based server implementations MUST satisfy the acceptance criteria before code review approval. Violations in authentication-critical code paths require architecture review escalation.
</enforcement>

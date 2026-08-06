# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Socket Based Authentication

These rules are ALWAYS ACTIVE for socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers, dual-stack IPv4/IPv6 socket binding operations, and structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.) in native Python and Emscripten/JavaScript interop contexts.

### Rules

- **R-SOCK-001** MUST: Socket-based authentication servers MUST use structured concurrency primitives (nursery.start or equivalent) to coordinate server lifecycle with shutdown events.
- **R-SOCK-002** MUST: Socket binding code MUST include retry logic handling `errno.EADDRINUSE` with up to 10 attempts to address port conflicts in CI environments.
- **R-SOCK-003** MUST: Dual-stack socket creation MUST use `socket.getaddrinfo(host, port, AF_UNSPEC, SOCK_STREAM, 0, AI_PASSIVE)` to enable both IPv4 and IPv6 binding when host is localhost.
- **R-SOCK-004** MUST: Socket options `TCP_NODELAY` and `SO_REUSEADDR` MUST be set immediately after socket creation and before bind().
- **R-SOCK-005** MUST: Hypercorn.trio.serve calls MUST be wrapped with `nursery.start(functools.partial(..., shutdown_trigger=event.wait))` to ensure shutdown coordination.
- **R-SOCK-006** SHOULD: Retry attempts MUST be logged to stderr for debugging purposes.
- **R-SOCK-007** SHOULD: In Emscripten contexts, cancellable request operations SHOULD use abort controller binding pattern with feature detection for abort controller availability.

### Verify

```bash
# Verify nursery.start usage with functools.partial for hypercorn lifecycle management
grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .

# Verify dual-stack socket configuration with AF_UNSPEC and TCP_NODELAY
grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .

# Verify retry logic for EADDRINUSE handling
grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'
```

**Accept when:**
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost
- Socket options TCP_NODELAY and SO_REUSEADDR are set before bind() operations
- Cancellable request operations in Emscripten context use abort controller binding pattern with feature detection
- Retry attempts are logged to stderr for debugging

<enforcement>
Clause Code MUST NOT skip or defer verification. All socket-based authentication server implementations MUST satisfy R-SOCK-001 through R-SOCK-007 before merge. Violations in authentication-critical code paths require architecture review escalation.
</enforcement>
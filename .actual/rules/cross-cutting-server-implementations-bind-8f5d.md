# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Server Implementations Bind

These rules are ALWAYS ACTIVE for socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers with structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.), and for request cancellation in browser environments via Emscripten/JavaScript interop.

### Rules

- **R-STRUCT-CONC-001** MUST: Server implementations MUST bind to both IPv4 and IPv6 address families when host is localhost to avoid connection delays from sequential address family attempts.
- **R-STRUCT-CONC-002** MUST: Wrap hypercorn.trio.serve calls with nursery.start(functools.partial(..., shutdown_trigger=event.wait)) to ensure shutdown coordination and structured resource cleanup.
- **R-STRUCT-CONC-003** MUST: Implement socket creation with socket.getaddrinfo(host, port, AF_UNSPEC, SOCK_STREAM, 0, AI_PASSIVE) to enable dual-stack binding.
- **R-STRUCT-CONC-004** MUST: Set socket options TCP_NODELAY and SO_REUSEADDR immediately after socket creation and before bind().
- **R-STRUCT-CONC-005** MUST: Wrap socket binding in retry loop catching OSError with errno.EADDRINUSE; log retry attempts to stderr for debugging (up to 10 attempts).
- **R-STRUCT-CONC-006** SHOULD: In Emscripten contexts, create abort controller and bind abort method before initiating fetch operations for cancellation support.

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
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management.
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost.
- Cancellable request operations in Emscripten context use abort controller binding pattern.
- Socket creation includes AF_UNSPEC in getaddrinfo call and sets TCP_NODELAY and SO_REUSEADDR options.

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All socket-based server implementations must demonstrate compliance with R-STRUCT-CONC-001 through R-STRUCT-CONC-005 before merge. Violations in authentication-critical code paths require architecture review escalation.
</enforcement>
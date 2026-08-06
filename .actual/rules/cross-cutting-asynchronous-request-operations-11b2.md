# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Asynchronous Request Operations

These rules are ALWAYS ACTIVE for socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers, dual-stack IPv4/IPv6 socket binding operations, structured concurrency frameworks (Trio, asyncio with TaskGroup), and request cancellation in browser environments via Emscripten/JavaScript interop.

### Rules

- **R-STRUCT-CONC-001** MUST: Asynchronous request operations MUST use abort controller patterns (js_abort_controller.abort.bind or equivalent) to enable cancellation propagation in Emscripten/browser contexts.
- **R-STRUCT-CONC-002** MUST: Socket-based server implementations MUST use nursery.start() or equivalent structured concurrency primitive for lifecycle management and graceful shutdown coordination.
- **R-STRUCT-CONC-003** MUST: Socket binding code MUST include retry logic handling errno.EADDRINUSE with up to 10 attempts to address port conflicts in CI environments.
- **R-STRUCT-CONC-004** MUST: Dual-stack socket creation MUST use socket.getaddrinfo(host, port, AF_UNSPEC, SOCK_STREAM, 0, AI_PASSIVE) to enable both IPv4 and IPv6 binding when host is localhost.
- **R-STRUCT-CONC-005** MUST: Socket options TCP_NODELAY and SO_REUSEADDR MUST be set immediately after socket creation and before bind().
- **R-STRUCT-CONC-006** SHOULD: Retry attempts for socket binding SHOULD be logged to stderr for debugging and monitoring purposes.

### Verify

```bash
# Verify nursery.start usage with hypercorn in server startup code
grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .

# Verify dual-stack socket configuration
grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .

# Verify retry logic for EADDRINUSE
grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'
```

**Accept when:**
- All socket-based server implementations use nursery.start() or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost
- Socket options TCP_NODELAY and SO_REUSEADDR are set immediately after socket creation
- Cancellable request operations in Emscripten context use abort controller binding pattern
- Retry attempts are logged to stderr for debugging

<enforcement>
Clause Code MUST NOT skip or defer verification. All socket-based server implementations and cancellable request operations MUST comply with these rules. Violations in authentication-critical code paths require architecture review escalation.
</enforcement>
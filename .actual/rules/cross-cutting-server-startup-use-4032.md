# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Server Startup Use

These rules are ALWAYS ACTIVE for socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers with structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.), dual-stack IPv4/IPv6 socket binding operations, and request cancellation in browser environments via Emscripten/JavaScript interop.

### Rules

- **R-STRUCT-CONC-001** SHOULD: Server startup SHOULD use functools.partial to bind configuration and shutdown triggers before passing to nursery.start

### Verify

```bash
# Verify nursery.start with functools.partial usage for hypercorn lifecycle management
grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .

# Verify dual-stack socket binding with AF_UNSPEC and TCP_NODELAY configuration
grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .

# Verify retry logic for EADDRINUSE handling
grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'
```

**Accept when:**
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost
- Cancellable request operations in Emscripten context use abort controller binding pattern
- Socket options TCP_NODELAY and SO_REUSEADDR are set immediately after socket creation and before bind()
- Socket creation uses socket.getaddrinfo(host, port, AF_UNSPEC, SOCK_STREAM, 0, AI_PASSIVE) for dual-stack binding

<enforcement>
Clause Code MUST NOT skip or defer verification. All socket-based server implementations MUST be reviewed against these rules during code review and CI checks. Violations in authentication-critical code paths require architecture review escalation.
</enforcement>
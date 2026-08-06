# Adopt Structured Concurrency with Nursery Pattern for Socket Lifecycle Management: Socket Creation Set

These rules are ALWAYS ACTIVE for all socket-based HTTP/HTTPS authentication servers using hypercorn or similar ASGI servers with structured concurrency frameworks (Trio, asyncio with TaskGroup, etc.), dual-stack IPv4/IPv6 socket binding operations, and request cancellation in browser environments via Emscripten/JavaScript interop.

### Rules

- **R-SOCKET-001** MUST: Socket creation MUST set TCP_NODELAY and SO_REUSEADDR options to optimize latency and enable rapid server restart.

### Verify

```bash
# Verify nursery.start usage with functools.partial for hypercorn lifecycle management
grep -r 'nursery\.start.*functools\.partial.*hypercorn' --include='*.py' .

# Verify dual-stack socket binding with AF_UNSPEC and TCP_NODELAY socket options
grep -r 'socket\.getaddrinfo.*AF_UNSPEC' --include='*.py' . && grep -r 'TCP_NODELAY' --include='*.py' .

# Verify retry logic for EADDRINUSE handling
grep -r 'errno\.EADDRINUSE' --include='*.py' . | grep -c 'retry\|range'
```

**Accept when:**
- All socket-based server implementations use nursery.start or equivalent structured concurrency primitive for lifecycle management
- Socket binding code includes retry logic handling EADDRINUSE and binds to both IPv4 and IPv6 when host is localhost
- Socket creation immediately sets TCP_NODELAY and SO_REUSEADDR before bind() operation
- Cancellable request operations in Emscripten context use abort controller binding pattern

<enforcement>
Clause R-SOCKET-001 verification is mandatory. Code review MUST confirm TCP_NODELAY and SO_REUSEADDR are set on all socket-based server implementations. CI MUST fail if dual-stack socket configuration is missing from new authentication server code paths.
</enforcement>
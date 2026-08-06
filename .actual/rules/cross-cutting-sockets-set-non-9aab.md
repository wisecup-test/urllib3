# Standardize Socket Binding and Core Library Usage for Network Server Components: Sockets Set Non

These rules are ALWAYS ACTIVE for network server components in dummyserver modules, socket binding and configuration logic, test server infrastructure requiring dual-stack IPv4/IPv6 support, Hypercorn configuration extensions, and browser-based fetch implementations using Emscripten.

### Rules

- **R-SOCK-001** SHOULD: Sockets SHOULD be set to non-blocking mode using sock.setblocking(False) for asynchronous server implementations.

### Verify

```bash
# Check for socket binding operations with explicit host and port pattern
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify socket configuration includes TCP_NODELAY and SO_REUSEADDR
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Check for EADDRINUSE error handling in retry logic
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'

# Check for non-blocking socket configuration
grep -r 'setblocking(False)' dummyserver/
```

**Accept when:**
- Socket binding operations use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules
- Sockets are configured with setblocking(False) for asynchronous operations
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo with AF_UNSPEC
- Retry logic includes exponential backoff up to 10 attempts with logging to stderr

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding implementations in scope MUST be checked against these rules during code review and CI pipeline execution.
</enforcement>
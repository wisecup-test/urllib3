# Standardize Socket Binding and Core Library Usage for Network Server Components: Socket Creation Support

These rules are ALWAYS ACTIVE for network server components in dummyserver modules, socket binding and configuration logic, test server infrastructure, Hypercorn configuration extensions, and browser-based fetch implementations using Emscripten.

### Rules

- **R-SOCKET-001** SHOULD: Socket creation SHOULD support dual-stack IPv4/IPv6 binding using socket.getaddrinfo with AF_UNSPEC or AF_INET6 family detection.

### Verify

```bash
# Check for explicit socket binding patterns
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify socket options are configured
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Check for EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'
```

**Accept when:**
- Socket binding operations use explicit `sock.bind((host, port))` pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo with appropriate address family detection
- Retry logic includes exponential backoff up to 10 attempts with logging of EADDRINUSE errors

<enforcement>
Claude Code MUST NOT skip or defer verification. Socket binding patterns MUST include required TCP_NODELAY and SO_REUSEADDR options. Retry logic MUST be present in new socket binding implementations. Cross-platform test execution on Windows, Linux, and macOS MUST validate socket binding behavior.
</enforcement>
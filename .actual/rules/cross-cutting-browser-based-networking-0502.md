# Standardize Socket Binding and Core Library Usage for Network Server Components: Browser Based Networking

These rules are ALWAYS ACTIVE for network server components in dummyserver modules, socket binding and configuration logic, test server infrastructure requiring dual-stack IPv4/IPv6 support, Hypercorn configuration extensions, and browser-based fetch implementations using Emscripten.

### Rules

- **R-SOCK-001** MUST: Implement socket binding retry logic with exponential backoff up to 10 attempts, logging each EADDRINUSE error to stderr for debugging.
- **R-SOCK-002** MUST: Use socket.getaddrinfo with AF_UNSPEC to discover both IPv4 and IPv6 addresses, binding to the same port number for both families.
- **R-SOCK-003** MUST: Configure sockets with TCP_NODELAY and SO_REUSEADDR immediately after creation and before binding.
- **R-SOCK-004** MUST: Set sock.set_inheritable(True) for sockets that need to be passed to child processes in server implementations.
- **R-SOCK-005** MAY: Browser-based networking components MAY use JavaScript interop with abort controllers bound via js_abort_controller.abort.bind() to maintain proper this context.
- **R-SOCK-006** MUST: Import and use core libraries (socket, sys, contextlib, errno, functools, logging, os, ssl, io, json, email.parser) for foundational networking and I/O operations.

### Verify

```bash
# Check for explicit socket binding patterns with host and port
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify TCP_NODELAY and SO_REUSEADDR configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Check for EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'
```

**Accept when:**
- Socket binding operations use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules
- Dual-stack IPv4/IPv6 support is implemented via socket.getaddrinfo with AF_UNSPEC
- Browser-based networking components properly bind JavaScript abort controllers using .bind()

<enforcement>
Claude Code MUST NOT skip or defer verification. Socket binding patterns MUST include required TCP_NODELAY and SO_REUSEADDR options. Retry logic MUST be present for new socket binding implementations. Cross-platform test execution on Windows, Linux, and macOS MUST validate socket binding behavior.
</enforcement>
# Standardize Socket Binding and Core Library Usage for Network Server Components: Core Library Imports

These rules are ALWAYS ACTIVE for all network server components in dummyserver modules and socket binding implementations that require dual-stack IPv4/IPv6 support and cross-platform compatibility.

### Rules

- **R-SOCKET-001** MUST: Core library imports MUST include socket, sys, and contextlib for foundational networking and resource management.

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
```

**Accept when:**
- Socket binding operations use explicit `sock.bind((host, port))` pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo with AF_UNSPEC
- Sockets are configured with `sock.set_inheritable(True)` when passed to child processes

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding implementations in network server components MUST include the required core library imports and follow the standardized binding patterns with retry logic and dual-stack support.
</enforcement>
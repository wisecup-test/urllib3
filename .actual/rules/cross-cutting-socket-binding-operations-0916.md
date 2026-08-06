# Standardize Socket Binding and Core Library Usage for Network Server Components: Socket Binding Operations

These rules are ALWAYS ACTIVE for all network server components in dummyserver modules and test server infrastructure requiring socket binding operations with dual-stack IPv4/IPv6 support.

### Rules

- **R-SOCK-001** MUST: Socket binding operations MUST use `sock.bind((host, port))` with explicit host and port parameters from the socket module.
- **R-SOCK-002** MUST: Socket configuration MUST include TCP_NODELAY and SO_REUSEADDR options immediately after socket creation and before binding.
- **R-SOCK-003** MUST: Socket binding retry logic MUST handle `errno.EADDRINUSE` errors with exponential backoff up to 10 attempts, logging each retry to stderr.
- **R-SOCK-004** MUST: Dual-stack IPv4/IPv6 support MUST use `socket.getaddrinfo()` with `AF_UNSPEC` to discover both address families and bind to the same port number for both.
- **R-SOCK-005** SHOULD: Sockets passed to child processes SHOULD have `sock.set_inheritable(True)` configured for proper process management.
- **R-SOCK-006** SHOULD: Core libraries (socket, sys, contextlib, errno, functools, logging, ssl, io, json) SHOULD be imported and available in network server modules.

### Verify

```bash
# Verify socket binding pattern with explicit host and port
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify socket options configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Verify EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'

# Verify dual-stack support with getaddrinfo
grep -r 'socket\.getaddrinfo' dummyserver/

# Verify set_inheritable usage for child processes
grep -r 'set_inheritable' dummyserver/
```

**Accept when:**
- Socket binding operations use explicit `sock.bind((host, port))` pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Retry logic implements exponential backoff with up to 10 attempts and logs to stderr
- Dual-stack IPv4/IPv6 support is implemented using `socket.getaddrinfo()` with AF_UNSPEC
- Core libraries (socket, sys, contextlib, errno, functools, logging, ssl, io, json) are imported and available in network server modules
- Sockets passed to child processes have `set_inheritable(True)` configured

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding operations in network server components MUST comply with R-SOCK-001 through R-SOCK-006. CI pipeline MUST fail if socket binding patterns do not include required TCP_NODELAY and SO_REUSEADDR options. Code review MUST block merge if retry logic is missing from new socket binding implementations.
</enforcement>
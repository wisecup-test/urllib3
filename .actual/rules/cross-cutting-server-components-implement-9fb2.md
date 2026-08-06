# Standardize Socket Binding and Core Library Usage for Network Server Components: Server Components Implement

These rules are ALWAYS ACTIVE for all network server components in dummyserver modules and test server infrastructure requiring socket binding and dual-stack IPv4/IPv6 support.

### Rules

- **R-SOCK-001** MUST: Server components MUST implement retry logic for socket binding to handle EADDRINUSE errors in concurrent test environments.
- **R-SOCK-002** MUST: Socket configuration MUST include TCP_NODELAY and SO_REUSEADDR options immediately after socket creation and before binding.
- **R-SOCK-003** MUST: Socket binding operations MUST use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors.
- **R-SOCK-004** SHOULD: Implement socket binding retry logic with exponential backoff up to 10 attempts, logging each EADDRINUSE error to stderr for debugging.
- **R-SOCK-005** SHOULD: Use socket.getaddrinfo with AF_UNSPEC to discover both IPv4 and IPv6 addresses, binding to the same port number for both families.
- **R-SOCK-006** SHOULD: Set sock.set_inheritable(True) for sockets that need to be passed to child processes in server implementations.
- **R-SOCK-007** MAY: For browser-based networking in Emscripten environments, bind JavaScript abort controllers using .bind() to maintain proper this context.

### Verify

```bash
# Check for explicit socket binding patterns with host and port
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify TCP_NODELAY and SO_REUSEADDR socket options are configured
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Confirm EADDRINUSE error handling for retry logic
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools; print("Core libs available")'
```

**Accept when:**
- Socket binding operations use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools) are imported and available in network server modules
- Retry logic is present with up to 10 attempts and logging of EADDRINUSE errors
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo with AF_UNSPEC

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding implementations in network server components MUST comply with R-SOCK-001 through R-SOCK-007. Violations detected by grep patterns or code review MUST block merge. Cross-platform test execution on Windows, Linux, and macOS MUST validate socket binding behavior.
</enforcement>
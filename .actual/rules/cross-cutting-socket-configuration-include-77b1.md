# Standardize Socket Binding and Core Library Usage for Network Server Components: Socket Configuration Include

These rules are ALWAYS ACTIVE for network server components in dummyserver modules, socket binding and configuration logic, test server infrastructure requiring dual-stack IPv4/IPv6 support, Hypercorn configuration extensions, and browser-based fetch implementations using Emscripten.

### Rules

- **R-SOCKET-001** MUST: Socket configuration MUST include TCP_NODELAY and SO_REUSEADDR options for optimal network performance and port reuse.
- **R-SOCKET-002** MUST: Socket binding operations MUST use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors.
- **R-SOCKET-003** MUST: Implement socket binding retry logic with exponential backoff up to 10 attempts, logging each EADDRINUSE error to stderr for debugging.
- **R-SOCKET-004** MUST: Use socket.getaddrinfo with AF_UNSPEC to discover both IPv4 and IPv6 addresses, binding to the same port number for both families.
- **R-SOCKET-005** MUST: Configure sockets with TCP_NODELAY and SO_REUSEADDR immediately after creation and before binding.
- **R-SOCKET-006** SHOULD: Set sock.set_inheritable(True) for sockets that need to be passed to child processes in server implementations.
- **R-SOCKET-007** SHOULD: For browser-based networking in Emscripten environments, bind JavaScript abort controllers using .bind() to maintain proper this context.
- **R-SOCKET-008** MUST: Core libraries (socket, sys, contextlib, errno, functools, logging, os, ssl, io, json, email.parser) MUST be imported and available in network server modules.

### Verify

```bash
# Check for explicit socket binding patterns with host and port
grep -r 'sock\.bind((.*host.*port))' dummyserver/ src/

# Verify TCP_NODELAY and SO_REUSEADDR configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/

# Check for EADDRINUSE error handling and retry logic
grep -r 'errno\.EADDRINUSE' dummyserver/

# Verify core libraries are available
python -c 'import socket, sys, contextlib, errno, functools, logging, os, ssl, io, json; print("Core libs available")'
```

**Accept when:**
- Socket binding operations use explicit sock.bind((host, port)) pattern with retry logic for EADDRINUSE errors
- Socket configuration includes TCP_NODELAY and SO_REUSEADDR options before binding
- Core libraries (socket, sys, contextlib, errno, functools, logging, os, ssl, io, json) are imported and available in network server modules
- Retry logic includes exponential backoff up to 10 attempts with stderr logging
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo with AF_UNSPEC
- Sockets are configured with TCP_NODELAY and SO_REUSEADDR immediately after creation

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding implementations in network server components MUST comply with these rules. CI pipeline MUST fail if socket binding patterns do not include required TCP_NODELAY and SO_REUSEADDR options. Code review MUST block merge if retry logic is missing from new socket binding implementations.
</enforcement>
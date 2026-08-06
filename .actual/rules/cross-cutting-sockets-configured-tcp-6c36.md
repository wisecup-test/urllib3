# Standardize Socket Binding via sock.bind() for Network Server Initialization: Sockets Configured Tcp

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** SHOULD: Sockets SHOULD be configured with TCP_NODELAY and SO_REUSEADDR options before binding to optimize connection handling and port reuse.
- **R-SOCK-002** MUST: All server socket initialization code MUST use sock.bind((host, port)) with explicit parameters.
- **R-SOCK-003** MUST: Socket binding code MUST include retry logic for EADDRINUSE errors with at least 10 attempts.
- **R-SOCK-004** MUST: Dual-stack IPv4/IPv6 support MUST be implemented using socket.getaddrinfo() with appropriate address family handling.
- **R-SOCK-005** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR) MUST be set immediately after socket creation and before binding.
- **R-SOCK-006** MUST: When binding to port 0 for automatic allocation, the assigned port MUST be extracted using sock.getsockname()[1] and reused for subsequent socket bindings.
- **R-SOCK-007** MUST: Socket binding operations MUST be wrapped in try-except blocks that specifically catch OSError with errno.EADDRINUSE, and log retry attempts to stderr.

### Verify

```bash
# Check for sock.bind() usage in server initialization
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Check for socket.getaddrinfo() usage for dual-stack support
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Check for EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'
```

**Accept when:**
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling
- Socket options TCP_NODELAY and SO_REUSEADDR are set before binding operations
- Port extraction via sock.getsockname()[1] is used when binding to port 0
- EADDRINUSE exceptions are caught and logged with retry information

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization MUST follow the sock.bind() pattern with explicit dual-stack support, retry logic, and proper socket option configuration.
</enforcement>
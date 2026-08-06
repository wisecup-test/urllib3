# Standardize Socket Binding via sock.bind() for Network Server Initialization: Socket Creation Handle

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** MUST: Socket creation MUST handle both IPv4 and IPv6 address families using socket.getaddrinfo() with socket.AF_UNSPEC or explicit family selection.
- **R-SOCK-002** MUST: All server socket initialization code MUST use sock.bind((host, port)) with explicit parameters.
- **R-SOCK-003** MUST: Socket binding code MUST include retry logic for EADDRINUSE errors with at least 10 attempts.
- **R-SOCK-004** MUST: Set socket options (TCP_NODELAY, SO_REUSEADDR) immediately after socket creation and before binding to ensure they take effect.
- **R-SOCK-005** MUST: When binding to port 0 for automatic allocation, extract the assigned port using sock.getsockname()[1] and reuse it for subsequent socket bindings to maintain consistent port across IPv4/IPv6.
- **R-SOCK-006** MUST: Wrap socket binding operations in try-except blocks that specifically catch OSError with errno.EADDRINUSE, and log retry attempts to stderr for debugging CI failures.
- **R-SOCK-007** SHOULD: Dual-stack IPv4/IPv6 support SHOULD be implemented using socket.getaddrinfo() with appropriate address family handling.

### Verify

```bash
# Check for sock.bind() usage in server initialization
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Check for socket.getaddrinfo() usage
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Check for EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'
```

**Accept when:**
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling
- Socket options (TCP_NODELAY, SO_REUSEADDR) are set immediately after socket creation and before binding
- Port extraction via sock.getsockname()[1] is used when binding to port 0
- OSError with errno.EADDRINUSE is specifically caught with retry attempts logged to stderr

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization MUST follow the sock.bind() pattern with explicit dual-stack support and EADDRINUSE retry logic. Code review rejection is required for violations. CI pipeline warnings must be issued for missing retry logic. Test failures on platforms without proper dual-stack binding must be addressed.
</enforcement>
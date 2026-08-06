# Standardize Socket Binding via sock.bind() for Network Server Initialization: Non Socket Network

These rules are ALWAYS ACTIVE for all network server initialization code, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** MUST: Non-socket network primitives (e.g., JavaScript fetch APIs in Emscripten contexts) MUST use framework-appropriate binding mechanisms such as abort_controller.abort.bind()
- **R-SOCK-002** MUST: All server socket initialization code use sock.bind((host, port)) with explicit parameters
- **R-SOCK-003** MUST: Socket binding code include retry logic for EADDRINUSE errors with at least 10 attempts
- **R-SOCK-004** MUST: Dual-stack IPv4/IPv6 support be implemented using socket.getaddrinfo() with appropriate address family handling
- **R-SOCK-005** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR) be set immediately after socket creation and before binding
- **R-SOCK-006** MUST: Socket binding operations be wrapped in try-except blocks that specifically catch OSError with errno.EADDRINUSE
- **R-SOCK-007** SHOULD: When binding to port 0 for automatic allocation, extract the assigned port using sock.getsockname()[1] and reuse it for subsequent socket bindings
- **R-SOCK-008** SHOULD: Retry attempts for port binding be logged to stderr for debugging CI failures

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
- Socket options are set before binding operations
- EADDRINUSE exceptions are caught and logged with retry information

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization must follow the sock.bind() pattern with explicit dual-stack support and EADDRINUSE retry logic. Code review rejection is required for violations.
</enforcement>
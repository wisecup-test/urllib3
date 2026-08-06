# Standardize Socket Binding via sock.bind() for Network Server Initialization: Server Configurations Use

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** MAY: Server configurations MAY use port 0 to request automatic port allocation from the operating system, then extract the assigned port via sock.getsockname()[1]
- **R-SOCK-002** MUST: All server socket initialization code use sock.bind((host, port)) with explicit parameters
- **R-SOCK-003** MUST: Socket binding code include retry logic for EADDRINUSE errors with at least 10 attempts
- **R-SOCK-004** MUST: Dual-stack IPv4/IPv6 support be implemented using socket.getaddrinfo() with appropriate address family handling
- **R-SOCK-005** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR) be set immediately after socket creation and before binding
- **R-SOCK-006** MUST: Socket binding operations be wrapped in try-except blocks that specifically catch OSError with errno.EADDRINUSE
- **R-SOCK-007** MUST: Retry attempts for socket binding be logged to stderr for debugging CI failures
- **R-SOCK-008** MUST: Socket cleanup be ensured in exception paths using context managers

### Verify

```bash
# Check for sock.bind() usage in server initialization
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Check for socket.getaddrinfo() usage for dual-stack support
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Check for EADDRINUSE error handling
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'

# Verify retry logic exists in socket binding code
grep -r 'for.*in.*range' dummyserver/ --include='*.py' -A 5 | grep -E '(bind|EADDRINUSE)'
```

**Accept when:**
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling
- Socket options are set before binding operations
- EADDRINUSE exceptions are caught and logged with retry information
- Socket cleanup uses context managers or try-finally blocks

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns in server initialization code. Code review rejection is required for server socket initialization that does not follow the sock.bind() pattern with proper error handling and dual-stack support.
</enforcement>
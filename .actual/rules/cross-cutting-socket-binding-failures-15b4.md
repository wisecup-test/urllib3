# Standardize Socket Binding via sock.bind() for Network Server Initialization: Socket Binding Failures

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** MUST: Socket binding failures with errno.EADDRINUSE MUST implement retry logic with a minimum of 10 attempts before raising OSError.

### Verify

```bash
# Check for sock.bind() usage in server initialization
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Check for socket.getaddrinfo() usage for dual-stack support
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Check for errno.EADDRINUSE handling
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'
```

**Accept when:**
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling
- Socket options (TCP_NODELAY, SO_REUSEADDR) are set immediately after socket creation and before binding
- Socket binding operations are wrapped in try-except blocks that specifically catch OSError with errno.EADDRINUSE
- Retry attempts are logged to stderr for debugging CI failures

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization must follow the sock.bind() pattern with explicit EADDRINUSE retry logic before code is accepted.
</enforcement>
# Standardize Socket Binding via sock.bind() for Network Server Initialization: Socket Binding Operations

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** MUST: Socket binding operations MUST use sock.bind((host, port)) with explicit host and port parameters for all TCP server socket initialization.

### Verify

```bash
# Check for sock.bind() usage in server initialization code
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Verify socket.getaddrinfo() is used for dual-stack support
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Confirm EADDRINUSE retry logic is implemented
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'
```

**Accept when:**
- All server socket initialization code uses sock.bind((host, port)) with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using socket.getaddrinfo() with appropriate address family handling
- Socket options (TCP_NODELAY, SO_REUSEADDR) are set immediately after socket creation and before binding
- Assigned ports from port 0 allocation are extracted using sock.getsockname()[1] and reused for subsequent bindings
- Socket binding operations are wrapped in try-except blocks that specifically catch OSError with errno.EADDRINUSE

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization must follow the sock.bind((host, port)) pattern with explicit parameters, retry logic for port contention, and dual-stack support via socket.getaddrinfo().
</enforcement>
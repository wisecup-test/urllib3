# Standardize Socket Binding via sock.bind() for Network Server Initialization: Socket Initialization Code

These rules are ALWAYS ACTIVE for all TCP server socket initialization in dummyserver components, custom socket creation methods in Hypercorn configuration classes, socket binding operations in test server infrastructure, and network primitive lifecycle management in Emscripten/JavaScript bridge code.

### Rules

- **R-SOCK-001** SHOULD: Socket initialization code SHOULD set blocking mode to False and inheritable flag to True for async framework integration.

### Verify

```bash
# Verify sock.bind() usage in server initialization
grep -r 'sock\.bind(' dummyserver/ src/ --include='*.py' | grep -v test

# Verify socket.getaddrinfo() for dual-stack support
grep -r 'socket\.getaddrinfo' dummyserver/ src/ --include='*.py'

# Verify EADDRINUSE retry logic
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py'
```

**Accept when:**
- All server socket initialization code uses `sock.bind((host, port))` with explicit parameters
- Socket binding code includes retry logic for EADDRINUSE errors with at least 10 attempts
- Dual-stack IPv4/IPv6 support is implemented using `socket.getaddrinfo()` with appropriate address family handling
- Socket options (TCP_NODELAY, SO_REUSEADDR) are set immediately after socket creation and before binding
- Blocking mode is set to False and inheritable flag is set to True for async framework integration

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding patterns. All server socket initialization must follow the sock.bind() pattern with explicit dual-stack support and EADDRINUSE retry logic.
</enforcement>
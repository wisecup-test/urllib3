# Standardize Socket Lifecycle Management Through Custom Config Classes: Socket Binding Operations

These rules are ALWAYS ACTIVE for server configuration classes, socket creation and binding logic in test infrastructure, proxy implementations requiring connection forwarding, and browser-based fetch implementations using abort controllers.

### Rules

- **R-SOCKET-001** MUST: Socket binding operations MUST implement retry logic with at least 10 attempts to handle EADDRINUSE errors in CI environments.

### Verify

```bash
# Verify custom socket lifecycle management implementations
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is present
grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'

# Verify dual-stack socket binding for localhost
grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'

# Verify socket option configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All server configuration classes override framework socket creation methods and implement retry logic for EADDRINUSE
- Socket creation for localhost binds both IPv4 and IPv6 addresses to the same port
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently set before binding across all implementations
- Public API contracts for socket lifecycle methods (create_sockets, bind, close, readable, writable, connect, start_forward) are documented and exposed in server configuration classes
- Retry logic logs all attempts with errno details and fails after 10 attempts with clear error messages

<enforcement>
Clause Code MUST NOT skip or defer verification. CI integration tests MUST verify dual-stack socket binding on Windows and Linux. Code review MUST block merge if new server configurations do not override create_sockets with documented rationale. Test failures on Windows due to IPv6 timeouts MUST trigger automatic review of socket binding implementation.
</enforcement>
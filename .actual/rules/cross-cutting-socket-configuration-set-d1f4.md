# Standardize Socket Lifecycle Management Through Custom Config Classes: Socket Configuration Set

These rules are ALWAYS ACTIVE for server configuration classes, socket creation and binding logic in test infrastructure, proxy implementations requiring connection forwarding, and browser-based fetch implementations using abort controllers.

### Rules

- **R-SOCKET-001** MUST: Socket configuration MUST set TCP_NODELAY and SO_REUSEADDR options before binding to optimize connection handling.
- **R-SOCKET-002** MUST: Server configuration classes extending framework config (hypercorn.Config, ASGI applications) MUST override framework socket creation methods and implement retry logic for EADDRINUSE errors.
- **R-SOCKET-003** MUST: Socket creation for localhost MUST bind both IPv4 and IPv6 addresses to the same port using socket.getaddrinfo with AF_UNSPEC or AF_INET6.
- **R-SOCKET-004** MUST: Socket creation MUST be wrapped in a retry loop (10 attempts) catching OSError with errno.EADDRINUSE, logging each retry to stderr.
- **R-SOCKET-005** MUST: Socket lifecycle methods (create_sockets, bind, close, readable, writable) and connection methods (connect, start_forward) MUST be exposed as public API contracts with docstrings.
- **R-SOCKET-006** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) MUST be set before binding across all implementations.

### Verify

```bash
# Verify custom socket creation overrides exist
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is implemented
grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'

# Verify dual-stack socket resolution
grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'

# Verify socket option configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All server configuration classes override framework socket creation methods and implement retry logic for EADDRINUSE
- Socket creation for localhost binds both IPv4 and IPv6 addresses to the same port
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently set before binding across all implementations
- Public API contracts for socket lifecycle methods are documented and exposed in server configuration classes
- Retry logic logs all attempts with errno details and fails after 10 attempts with clear error messages
- Socket lifecycle methods include docstrings describing their role as extension points

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket configuration implementations MUST satisfy R-SOCKET-001 through R-SOCKET-006 before code review approval. CI integration tests MUST verify dual-stack socket binding on Windows and Linux. Code review MUST block merge if new server configurations do not override create_sockets with documented rationale.
</enforcement>
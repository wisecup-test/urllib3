# Standardize Socket Lifecycle Management Through Custom Config Classes: Implementations Use Abort

These rules are ALWAYS ACTIVE for server configuration classes, socket lifecycle implementations, test infrastructure (dummyserver), proxy implementations, and browser-based fetch implementations using abort controllers.

### Rules

- **R-SOCKET-001** MAY: Implementations MAY use abort controllers (js_abort_controller.abort.bind) for cancellation in browser-based environments.
- **R-SOCKET-002** MUST: Server configuration classes extending framework config (hypercorn.Config, ASGI applications) override framework socket creation methods and implement retry logic for EADDRINUSE errors.
- **R-SOCKET-003** MUST: Socket creation for localhost bind both IPv4 and IPv6 addresses to the same port using socket.getaddrinfo with AF_UNSPEC or AF_INET6.
- **R-SOCKET-004** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) be consistently set before binding across all implementations.
- **R-SOCKET-005** MUST: Retry logic wrap socket creation in a loop (10 attempts) catching OSError with errno.EADDRINUSE, logging each retry to stderr.
- **R-SOCKET-006** MUST: Public API contracts for socket lifecycle methods (create_sockets, bind, close, readable, writable, connect, start_forward) be documented and exposed in server configuration classes.

### Verify

```bash
# Verify custom socket lifecycle management implementations
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is present
grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'

# Verify dual-stack socket binding configuration
grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'

# Verify socket option configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All server configuration classes override framework socket creation methods and implement retry logic for EADDRINUSE
- Socket creation for localhost binds both IPv4 and IPv6 addresses to the same port
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently set before binding across all implementations
- Public API contracts for socket lifecycle methods are documented and exposed in server configuration classes
- CI integration tests verify dual-stack socket binding on Windows and Linux
- Retry logic logs all attempts with errno details and fails after 10 attempts with clear error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket lifecycle implementations MUST satisfy R-SOCKET-002 through R-SOCKET-006. Violations block merge and trigger automatic review of socket binding implementation.
</enforcement>
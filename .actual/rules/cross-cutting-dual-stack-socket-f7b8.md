# Standardize Socket Lifecycle Management Through Custom Config Classes: Dual Stack Socket

These rules are ALWAYS ACTIVE for all server configuration classes, socket creation and binding logic in test infrastructure, proxy implementations requiring connection forwarding, and browser-based fetch implementations using abort controllers.

### Rules

- **R-SOCKET-001** MUST: Dual-stack socket creation MUST bind both IPv4 and IPv6 addresses when host is localhost to prevent Happy Eyeballs timeout delays.
- **R-SOCKET-002** MUST: Server configuration classes extending framework config (hypercorn.Config, ASGI applications) MUST override framework socket creation methods and implement retry logic for EADDRINUSE errors.
- **R-SOCKET-003** MUST: Socket creation for localhost MUST bind both IPv4 and IPv6 addresses to the same port.
- **R-SOCKET-004** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) MUST be consistently set before binding across all implementations.
- **R-SOCKET-005** MUST: Retry logic for EADDRINUSE errors MUST be implemented with up to 10 attempts, logging each retry to stderr with errno details.
- **R-SOCKET-006** MUST: Public API contracts for socket lifecycle methods (create_sockets, bind, close, readable, writable, connect, start_forward) MUST be documented and exposed in server configuration classes.
- **R-SOCKET-007** SHOULD: Use socket.getaddrinfo with AF_UNSPEC or AF_INET6 to resolve both IPv4 and IPv6 addresses for localhost.

### Verify

```bash
# Verify custom socket creation overrides exist
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is implemented
grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'

# Verify dual-stack address resolution
grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'

# Verify socket options are configured
grep -r 'TCP_NODELAY\|SO_REUSEADDR' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All server configuration classes override framework socket creation methods and implement retry logic for EADDRINUSE.
- Socket creation for localhost binds both IPv4 and IPv6 addresses to the same port.
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently set before binding across all implementations.
- Public API contracts for socket lifecycle methods are documented and exposed in server configuration classes.
- CI integration tests verify dual-stack socket binding on Windows and Linux.
- Retry logic logs all attempts with errno details and fails after 10 attempts with clear error messages.

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket lifecycle implementations MUST be reviewed against these rules before merge. CI MUST fail if socket creation does not implement retry logic or dual-stack binding. Code review MUST block merge if new server configurations do not override create_sockets with documented rationale.
</enforcement>
# Standardize Socket Lifecycle Management Through Custom Config Classes: Public Contracts Expose

These rules are ALWAYS ACTIVE for server configuration classes, socket lifecycle implementations, and test infrastructure that manages socket creation, binding, and lifecycle across multiple server implementations (Hypercorn, ASGI proxy, dummyserver).

### Rules

- **R-SOCKET-001** SHOULD: Public API contracts SHOULD expose socket lifecycle methods (create_sockets, bind, close, readable, writable) as documented extension points.
- **R-SOCKET-002** MUST: Server configuration classes extending framework config (hypercorn.Config, ASGI applications) MUST override framework socket creation methods and implement retry logic for EADDRINUSE errors.
- **R-SOCKET-003** MUST: Socket creation for localhost MUST bind both IPv4 and IPv6 addresses to the same port (dual-stack binding).
- **R-SOCKET-004** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) MUST be consistently set before binding across all implementations.
- **R-SOCKET-005** MUST: Retry logic for EADDRINUSE MUST implement up to 10 attempts with logging of each retry attempt to stderr.
- **R-SOCKET-006** SHOULD: Socket lifecycle methods (create_sockets, bind, connect, start_forward, close, readable, writable) SHOULD be documented as stable APIs with semantic versioning guarantees.

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
- Retry logic implements up to 10 attempts with logging of each retry to stderr
- Socket lifecycle methods include docstrings describing their role as extension points

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket lifecycle implementations MUST be reviewed against these rules before acceptance. CI integration tests MUST verify dual-stack socket binding on Windows and Linux. Code review MUST block merge if new server configurations do not override create_sockets with documented rationale.
</enforcement>
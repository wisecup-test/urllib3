# Standardize Socket Lifecycle Management Through Custom Config Classes: Connection Management Methods

These rules are ALWAYS ACTIVE for server configuration classes, socket lifecycle implementations, proxy forwarding code, and test infrastructure that manages socket creation, binding, and connection management across Hypercorn, ASGI, and custom server implementations.

### Rules

- **R-SOCK-001** SHOULD: Connection management methods (connect, start_forward, absolute_uri) SHOULD be exposed as public contracts for proxy and forwarding implementations.
- **R-SOCK-002** MUST: Server configuration classes extending framework config (hypercorn.Config, ASGI applications) MUST override framework socket creation methods and implement retry logic for EADDRINUSE errors.
- **R-SOCK-003** MUST: Socket creation for localhost MUST bind both IPv4 and IPv6 addresses to the same port using socket.getaddrinfo with AF_UNSPEC or AF_INET6.
- **R-SOCK-004** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) MUST be consistently set before binding across all server implementations.
- **R-SOCK-005** MUST: Socket creation MUST be wrapped in a retry loop (10 attempts) catching OSError with errno.EADDRINUSE, logging each retry to stderr.
- **R-SOCK-006** MUST: Socket lifecycle methods (create_sockets, bind, close, readable, writable) and connection methods (connect, start_forward) MUST be exposed as public API contracts with docstrings.

### Verify

```bash
# Verify custom socket lifecycle management implementations exist
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is implemented
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
- Retry logic logs all attempts with errno details and fails after 10 attempts with clear error messages
- Connection management methods are documented as stable public APIs with semantic versioning guarantees

<enforcement>
Clause Code MUST NOT skip or defer verification. All socket lifecycle implementations MUST satisfy R-SOCK-002 through R-SOCK-006. Violations detected by grep patterns or test failures on Windows due to IPv6 timeouts trigger automatic code review. New server configurations without documented create_sockets overrides MUST be blocked at merge.
</enforcement>
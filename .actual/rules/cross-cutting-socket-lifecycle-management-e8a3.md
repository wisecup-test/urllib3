# Standardize Socket Lifecycle Management Through Custom Config Classes: Socket Lifecycle Management

These rules are ALWAYS ACTIVE for server configuration classes, socket lifecycle implementations, test infrastructure (dummyserver), and proxy implementations requiring connection forwarding.

### Rules

- **R-SOCK-001** MUST: Socket lifecycle management MUST be encapsulated in custom configuration classes that override framework defaults (e.g., Config.create_sockets, Config._create_urllib3_sockets).
- **R-SOCK-002** MUST: Socket creation for localhost MUST bind both IPv4 and IPv6 addresses to the same port using dual-stack binding.
- **R-SOCK-003** MUST: Socket creation MUST implement retry logic for EADDRINUSE errors with up to 10 attempts, logging each retry to stderr.
- **R-SOCK-004** MUST: Socket options (TCP_NODELAY, SO_REUSEADDR, setblocking(False), set_inheritable(True)) MUST be consistently set before binding across all implementations.
- **R-SOCK-005** MUST: Public API contracts for socket lifecycle methods (create_sockets, bind, close, readable, writable, connect, start_forward) MUST be documented with docstrings and exposed in server configuration classes.
- **R-SOCK-006** SHOULD: Use socket.getaddrinfo with AF_UNSPEC or AF_INET6 to resolve both IPv4 and IPv6 addresses for localhost.
- **R-SOCK-007** SHOULD: Log all retry attempts with errno details to stderr and fail after 10 attempts with clear error messages.

### Verify

```bash
# Verify custom socket creation overrides exist
grep -r 'def create_sockets' --include='*.py' | grep -v '__pycache__'

# Verify EADDRINUSE retry logic is implemented
grep -r 'errno.EADDRINUSE' --include='*.py' | grep -v '__pycache__'

# Verify dual-stack socket binding
grep -r 'socket.getaddrinfo.*AF_INET6\|AF_UNSPEC' --include='*.py' | grep -v '__pycache__'

# Verify socket options are configured
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
Claude Code MUST NOT skip or defer verification. All socket lifecycle implementations MUST satisfy R-SOCK-001 through R-SOCK-007. Code review MUST block merge if new server configurations do not override create_sockets with documented rationale. CI MUST fail if socket creation does not implement retry logic or dual-stack binding.
</enforcement>
# Standardize Socket Binding for Test Server Authentication Providers: Sockets Configure Tcp

These rules are ALWAYS ACTIVE for all test server socket creation and binding in dummyserver modules, Hypercorn configuration, socket-based authentication provider setup, and IPv4/IPv6 dual-stack socket allocation.

### Rules

- **R-SOCK-001** SHOULD: Sockets SHOULD configure TCP_NODELAY and SO_REUSEADDR options for optimal connection handling.

### Verify

```bash
# Verify socket.bind() usage with (host, port) tuple parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/

# Verify socket.getaddrinfo() with AI_PASSIVE flag for address resolution
grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'

# Verify TCP_NODELAY and SO_REUSEADDR socket option configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l
```

**Accept when:**
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count
- TCP_NODELAY and SO_REUSEADDR options are configured on sockets in test server implementations

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations block CI pipeline and code review merge until socket binding patterns match required structure and retry logic is present in all new server implementations.
</enforcement>
# Standardize Socket Binding for Test Server Authentication Providers: Sockets Set Non

These rules are ALWAYS ACTIVE for all test server socket creation and binding operations in dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup.

### Rules

- **R-SOCK-001** MUST: Sockets MUST be set to non-blocking mode (setblocking(False)) before binding

### Verify

```bash
# Check for socket.bind() usage patterns
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/

# Verify socket.getaddrinfo() with AI_PASSIVE flag usage
grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'

# Count TCP_NODELAY and SO_REUSEADDR socket option configurations
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l

# Verify non-blocking socket mode is set
grep -r 'setblocking(False)' dummyserver/
```

**Accept when:**
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count
- Non-blocking socket mode (setblocking(False)) is set before binding operations

<enforcement>
Clause Code MUST NOT skip or defer verification. All socket binding implementations in scope MUST comply with R-SOCK-001 before merge.
</enforcement>
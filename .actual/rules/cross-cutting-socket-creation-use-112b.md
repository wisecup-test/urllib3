# Standardize Socket Binding for Test Server Authentication Providers: Socket Creation Use

These rules are ALWAYS ACTIVE for all test server socket creation in dummyserver modules, Hypercorn configuration socket binding, and socket-based authentication provider setup across IPv4 and IPv6 dual-stack socket allocation.

### Rules

- **R-SOCKET-001** MUST: Socket creation MUST use socket.getaddrinfo() to resolve host addresses and support both IPv4 and IPv6.

### Verify

```bash
# Check for sock.bind() usage with (host, port) tuple parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/

# Verify socket.getaddrinfo() with AI_PASSIVE flag is used
grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'

# Verify socket option configuration (TCP_NODELAY, SO_REUSEADDR)
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l
```

**Accept when:**
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently configured across implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if socket binding patterns do not match required structure. CI pipeline MUST fail if retry logic is missing from new server implementations. Test failures on Windows MUST trigger investigation of IPv6 binding issues.
</enforcement>
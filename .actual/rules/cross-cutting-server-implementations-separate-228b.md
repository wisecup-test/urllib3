# Standardize Socket Binding for Test Server Authentication Providers: Server Implementations Separate

These rules are ALWAYS ACTIVE for test server socket binding implementations across dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup.

### Rules

- **R-SOCK-001** MAY: Server implementations MAY separate secure_sockets and insecure_sockets based on SSL configuration

### Verify

```bash
# Verify socket.bind() usage with (host, port) tuple parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/

# Verify socket.getaddrinfo() with AI_PASSIVE flag for address resolution
grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'

# Verify socket option configuration (TCP_NODELAY, SO_REUSEADDR)
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l
```

**Accept when:**
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count
- Dual-stack IPv4/IPv6 binding is supported without sequential connection delays
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently configured across implementations

<enforcement>
Clause Code MUST NOT skip or defer verification of socket binding patterns. Code review MUST block merges if retry logic is missing from new server implementations. CI pipeline MUST fail if socket binding patterns do not match required structure. Test failures on Windows MUST trigger investigation of IPv6 binding issues.
</enforcement>
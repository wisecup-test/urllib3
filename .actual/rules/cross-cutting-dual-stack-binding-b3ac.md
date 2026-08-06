# Standardize Socket Binding for Test Server Authentication Providers: Dual Stack Binding

These rules are ALWAYS ACTIVE for all test server socket binding implementations in dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup across IPv4 and IPv6 dual-stack socket allocation.

### Rules

- **R-DUALSTACK-001** SHOULD: Dual-stack binding SHOULD reuse the same port number across IPv4 and IPv6 addresses.

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
- Dual-stack binding reuses the same port number across IPv4 and IPv6 addresses
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently configured across implementations

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review of test server implementations is mandatory. Automated grep patterns in CI must check for socket.bind() usage. Integration tests must validate dual-stack binding and port allocation. CI pipeline fails if socket binding patterns do not match required structure. Code review blocks merge if retry logic is missing from new server implementations.
</enforcement>
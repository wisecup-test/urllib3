# Standardize Socket Binding for Test Server Authentication Providers: Socket Binding Use

These rules are ALWAYS ACTIVE for all test server socket binding implementations in dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup across IPv4 and IPv6 dual-stack socket allocation.

### Rules

- **R-SOCKET-001** MUST: Socket binding MUST use socket.bind() with explicit (host, port) tuple parameters.

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
- Socket option configuration (TCP_NODELAY, SO_REUSEADDR) is present in implementations
- Retry logic wraps socket binding in try-except blocks catching OSError with errno.EADDRINUSE

<enforcement>
Claude Code MUST NOT skip or defer verification. All socket binding implementations MUST conform to R-SOCKET-001. Code review MUST block merge if retry logic is missing from new server implementations. CI pipeline MUST fail if socket binding patterns do not match required structure.
</enforcement>
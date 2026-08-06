# Standardize Socket Binding for Test Server Authentication Providers: Socket Binding Implement

These rules are ALWAYS ACTIVE for all test server socket binding implementations in dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup.

### Rules

- **R-SOCKET-001** SHOULD: Socket binding SHOULD implement retry logic (up to 10 attempts) to handle EADDRINUSE errors in concurrent environments.

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
- Socket options (TCP_NODELAY, SO_REUSEADDR) are consistently configured across implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if retry logic is missing from new server implementations. CI pipeline MUST fail if socket binding patterns do not match required structure.
</enforcement>
# Standardize Socket Binding for Test Server Authentication Providers: Server Implementations Support

These rules are ALWAYS ACTIVE for all test server implementations in the dummyserver modules, Hypercorn configuration, and socket-based authentication provider setup across IPv4 and IPv6 dual-stack socket allocation.

### Rules

- **R-SOCKET-001** MUST: Server implementations MUST support port 0 allocation to obtain random available ports.
- **R-SOCKET-002** MUST: Use socket.getaddrinfo() with socket.AI_PASSIVE flag to resolve bind addresses for both IPv4 and IPv6.
- **R-SOCKET-003** MUST: Extract port number from first bound socket using sock.getsockname()[1] and reuse for subsequent dual-stack bindings.
- **R-SOCKET-004** MUST: Wrap socket binding in try-except blocks catching OSError with errno.EADDRINUSE for retry logic.
- **R-SOCKET-005** SHOULD: Set socket.set_inheritable(True) to allow socket passing to child processes if needed.
- **R-SOCKET-006** SHOULD: Log retry attempts to stderr for debugging CI failures: print(f'Retrying binding to {bind} after EADDRINUSE', file=sys.stderr).
- **R-SOCKET-007** SHOULD: Configure socket options including TCP_NODELAY and SO_REUSEADDR to optimize connection handling and enable rapid socket reuse.

### Verify

```bash
# Verify socket.bind() usage with (host, port) tuple parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/

# Verify socket.getaddrinfo() with AI_PASSIVE flag usage
grep -r 'socket\.getaddrinfo' dummyserver/ | grep -c 'AI_PASSIVE'

# Verify TCP_NODELAY and SO_REUSEADDR socket option configuration
grep -r 'TCP_NODELAY\|SO_REUSEADDR' dummyserver/ | wc -l

# Verify retry logic for EADDRINUSE errors
grep -r 'EADDRINUSE\|errno\.EADDRINUSE' dummyserver/
```

**Accept when:**
- All test server implementations use socket.bind() with (host, port) tuple parameters
- Socket creation uses socket.getaddrinfo() with AI_PASSIVE flag for address resolution
- At least one implementation includes retry logic for EADDRINUSE errors with configurable attempt count
- Socket options (TCP_NODELAY, SO_REUSEADDR) are configured consistently across implementations
- Port 0 allocation is supported for obtaining random available ports

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review must validate socket binding patterns match required structure. CI pipeline must fail if socket binding patterns do not match. Test failures on Windows indicate IPv6 binding issues requiring remediation.
</enforcement>
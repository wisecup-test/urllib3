# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Socket Binding Implementations

These rules are ALWAYS ACTIVE for all test server socket binding implementations in `dummyserver/` and related test infrastructure files that create HTTP/HTTPS endpoints using Python's socket module.

### Rules

- **R-SOCK-001** MUST: Socket binding implementations MUST implement retry logic with at least 10 attempts when encountering `errno.EADDRINUSE` to handle port conflicts in CI environments.

### Verify

```bash
# Check for socket.bind patterns with host and port parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Check for dual-stack binding via socket.getaddrinfo with AF_UNSPEC or AF_INET6
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Check for EADDRINUSE retry logic with at least 10 attempts
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in `dummyserver/` use `socket.bind((host, port))` with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through `socket.getaddrinfo` results with `AF_UNSPEC` or `AF_INET6`
- Retry logic with `errno.EADDRINUSE` handling is present in socket binding code with at least 10 retry attempts

<enforcement>
Clause Code MUST NOT skip or defer verification of socket binding implementations against these rules. All test server socket binding code must be reviewed for compliance with retry logic and dual-stack support requirements before merge.
</enforcement>
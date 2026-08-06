# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Socket Binding Implementations

These rules are ALWAYS ACTIVE for all test server socket binding implementations in `dummyserver/` and related test infrastructure files that create network endpoints for HTTP/HTTPS test servers.

### Rules

- **R-SOCK-001** SHOULD: Socket binding implementations SHOULD set sockets to non-blocking mode using `sock.setblocking(False)` after successful binding.

### Verify

```bash
# Verify socket binding patterns are present
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Verify dual-stack binding with getaddrinfo
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Verify retry logic for EADDRINUSE with 10 attempts
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in `dummyserver/` use `socket.bind((host, port))` with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through `socket.getaddrinfo` results with `AF_UNSPEC` or `AF_INET6`
- Retry logic with `errno.EADDRINUSE` handling is present in socket binding code with at least 10 retry attempts
- Sockets are configured with `sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)` and `sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)` before binding
- After successful binding, `sock.setblocking(False)` and `sock.set_inheritable(True)` are called

<enforcement>
Clause Code MUST NOT skip or defer verification of socket binding implementations. All test server socket binding code MUST follow dual-stack patterns with retry logic and non-blocking mode configuration. Violations detected by grep-based verification in CI pipeline MUST block merge. Platform-specific exceptions (IPv6 unavailable, WebAssembly environments) require documented justification and architecture team approval.
</enforcement>
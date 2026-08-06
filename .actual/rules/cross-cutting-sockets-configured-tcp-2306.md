# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Sockets Configured Tcp

These rules are ALWAYS ACTIVE for all test server socket binding implementations in `dummyserver/` and related test infrastructure files that create TCP sockets for HTTP/HTTPS test endpoints.

### Rules

- **R-SOCK-001** SHOULD: Sockets SHOULD be configured with TCP_NODELAY and SO_REUSEADDR options before binding to optimize connection handling.
- **R-SOCK-002** SHOULD: Sockets SHOULD use `socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM, 0, socket.AI_PASSIVE)` to retrieve all available address families for dual-stack IPv4/IPv6 binding.
- **R-SOCK-003** SHOULD: Socket binding SHOULD be wrapped in a retry loop that catches `OSError` with `errno.EADDRINUSE`, logs retry attempts to stderr, and attempts up to 10 times before raising the exception.
- **R-SOCK-004** SHOULD: After binding the first socket with port 0, the allocated port SHOULD be extracted using `sock.getsockname()[1]` and reused for subsequent address family bindings to ensure consistent port allocation across dual-stack configurations.
- **R-SOCK-005** SHOULD: Sockets SHOULD be set to non-blocking mode with `sock.setblocking(False)` and inheritable with `sock.set_inheritable(True)` after successful binding.

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
- Sockets are configured with `TCP_NODELAY` and `SO_REUSEADDR` options before binding
- Port allocation is consistent across IPv4 and IPv6 sockets by reusing the port from the first successful binding

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding implementations. All test server socket creation MUST follow dual-stack binding patterns with retry logic and socket option configuration as specified in these rules.
</enforcement>
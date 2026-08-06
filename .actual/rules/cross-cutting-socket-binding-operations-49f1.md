# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Socket Binding Operations

These rules are ALWAYS ACTIVE for all test server socket binding implementations in the dummyserver/ directory and related test infrastructure code that creates network endpoints for HTTP/HTTPS test servers.

### Rules

- **R-SOCK-001** MUST: Socket binding operations MUST use `socket.bind((host, port))` with explicit host and port parameters for all test server implementations.
- **R-SOCK-002** MUST: Use `socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM, 0, socket.AI_PASSIVE)` to retrieve all available address families for dual-stack binding support.
- **R-SOCK-003** MUST: After binding the first socket with port 0, extract the allocated port using `sock.getsockname()[1]` and reuse this port for subsequent address family bindings.
- **R-SOCK-004** MUST: Wrap socket binding in a retry loop that catches `OSError` with `errno.EADDRINUSE`, logs the retry attempt to stderr, and attempts up to 10 times before raising the exception.
- **R-SOCK-005** MUST: Configure sockets with `sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)` and `sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)` before binding.
- **R-SOCK-006** MUST: Set `sock.setblocking(False)` and `sock.set_inheritable(True)` after successful binding to ensure non-blocking operation and proper subprocess inheritance.
- **R-SOCK-007** SHOULD: Log each retry attempt with stderr output to provide visibility into binding failures and retry frequency for CI environment diagnostics.
- **R-SOCK-008** SHOULD: Validate port consistency after binding by checking `sock.getsockname()[1]` for all created sockets; fall back to separate ports if unified port allocation fails.

### Verify

```bash
# Verify socket.bind patterns with explicit host and port parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Verify dual-stack binding via getaddrinfo with AF_UNSPEC or AF_INET6
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Verify retry logic with errno.EADDRINUSE handling and 10 retry attempts
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in dummyserver/ use `socket.bind((host, port))` with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through `socket.getaddrinfo` results with `AF_UNSPEC` or `AF_INET6`
- Retry logic with `errno.EADDRINUSE` handling is present in socket binding code with at least 10 retry attempts
- Socket options for TCP_NODELAY and SO_REUSEADDR are configured before binding
- Non-blocking mode and inheritable flags are set after successful binding

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding implementations. All grep-based verification commands MUST pass before accepting socket binding code. Code review MUST block merge if new test server implementations do not follow dual-stack binding requirements. Test failures on Windows indicating IPv6 connection timeouts MUST trigger investigation of socket binding implementation compliance.
</enforcement>
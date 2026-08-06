# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Test Servers Binding

These rules are ALWAYS ACTIVE for all test server socket binding implementations in `dummyserver/hypercornserver.py`, `dummyserver/socketserver.py`, and related HTTP/HTTPS test endpoint socket creation code.

### Rules

- **R-BIND-001** MUST: Test servers binding to localhost MUST support dual-stack IPv4 and IPv6 by iterating through `socket.getaddrinfo()` results with `socket.AF_UNSPEC` or `socket.AF_INET6` family.
- **R-BIND-002** MUST: Socket binding operations MUST use `socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM, 0, socket.AI_PASSIVE)` to retrieve all available address families for the specified host.
- **R-BIND-003** MUST: After binding the first socket with port 0, extract the allocated port using `sock.getsockname()[1]` and reuse this port for subsequent address family bindings to ensure consistent endpoint addressing.
- **R-BIND-004** MUST: Socket binding MUST be wrapped in a retry loop that catches `OSError` with `errno.EADDRINUSE`, logs the retry attempt to stderr, and attempts up to 10 times before raising the exception.
- **R-BIND-005** MUST: Sockets MUST be configured with `sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)` and `sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)` before binding.
- **R-BIND-006** MUST: After successful binding, sockets MUST be set to non-blocking mode with `sock.setblocking(False)` and `sock.set_inheritable(True)` to ensure proper subprocess inheritance.

### Verify

```bash
# Check for socket.bind patterns in test server implementations
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Verify dual-stack binding with getaddrinfo
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Verify retry logic with EADDRINUSE handling
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in `dummyserver/` use `socket.bind((host, port))` with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through `socket.getaddrinfo()` results with `AF_UNSPEC` or `AF_INET6`
- Retry logic with `errno.EADDRINUSE` handling is present in socket binding code with at least 10 retry attempts
- Socket configuration includes `SO_REUSEADDR`, `TCP_NODELAY`, `setblocking(False)`, and `set_inheritable(True)` calls

<enforcement>
Claude Code MUST NOT skip or defer verification of these socket binding rules. All test server implementations MUST comply with dual-stack binding requirements to prevent IPv6 connection timeout failures on Windows and ensure consistent behavior across CI environments.
</enforcement>
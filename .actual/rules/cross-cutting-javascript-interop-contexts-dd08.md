# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: Javascript Interop Contexts

These rules are ALWAYS ACTIVE for test server socket binding implementations in `dummyserver/` and JavaScript interop contexts in WebAssembly/Emscripten environments.

### Rules

- **R-SOCKET-001** MUST: Use `socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM, 0, socket.AI_PASSIVE)` to retrieve all available address families for dual-stack IPv4/IPv6 binding.
- **R-SOCKET-002** MUST: Bind sockets using `sock.bind((host, port))` pattern with explicit host and port parameters across all test server implementations.
- **R-SOCKET-003** MUST: Implement retry logic that catches `OSError` with `errno.EADDRINUSE`, logs retry attempts to stderr, and attempts up to 10 times before raising the exception.
- **R-SOCKET-004** MUST: After binding the first socket with port 0, extract the allocated port using `sock.getsockname()[1]` and reuse this port for subsequent address family bindings.
- **R-SOCKET-005** MUST: Configure sockets with `sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)` and `sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)` before binding.
- **R-SOCKET-006** MUST: Set `sock.setblocking(False)` and `sock.set_inheritable(True)` after successful binding to ensure non-blocking operation and proper subprocess inheritance.
- **R-INTEROP-001** MAY: JavaScript interop contexts MAY use `bind()` method for function binding patterns (e.g., `js_abort_controller.abort.bind`) when integrating with browser or WebAssembly APIs.

### Verify

```bash
# Verify socket.bind pattern usage
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Verify dual-stack address family handling
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Verify EADDRINUSE retry logic with 10 attempts
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in `dummyserver/` use `socket.bind((host, port))` with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through `socket.getaddrinfo` results with `AF_UNSPEC` or `AF_INET6`
- Retry logic with `errno.EADDRINUSE` handling is present in socket binding code with at least 10 retry attempts
- Socket options for `TCP_NODELAY`, `SO_REUSEADDR`, `setblocking(False)`, and `set_inheritable(True)` are configured
- Port reuse across address families is validated by checking `sock.getsockname()[1]` consistency

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding implementations. All R-SOCKET rules MUST be verified before accepting changes to test server infrastructure. Violations detected by grep-based verification in CI pipeline MUST block merge. Platform-specific exceptions (EXC-001, EXC-002) require explicit documentation and architecture team approval.
</enforcement>
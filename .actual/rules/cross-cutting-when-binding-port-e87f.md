# Standardize Socket Binding with Dual-Stack IPv4/IPv6 Support for Test Server Infrastructure: When Binding Port

These rules are ALWAYS ACTIVE for all test server socket binding implementations in the dummyserver/ directory, including hypercornserver.py, socketserver.py, and any other modules that create HTTP/HTTPS test endpoints using socket.bind() operations.

### Rules

- **R-SOCKET-001** MUST: When binding to port 0 for random port allocation, implementations MUST reuse the allocated port number from the first socket for subsequent address family bindings to ensure consistent port across IPv4 and IPv6.

### Verify

```bash
# Check for socket.bind patterns with host and port parameters
grep -r 'sock\.bind((.*host.*port.*))' dummyserver/ --include='*.py'

# Check for dual-stack binding via socket.getaddrinfo with AF_UNSPEC or AF_INET6
grep -r 'socket\.getaddrinfo.*AF_UNSPEC\|AF_INET6' dummyserver/ --include='*.py'

# Check for retry logic with errno.EADDRINUSE handling and 10 retry attempts
grep -r 'errno\.EADDRINUSE' dummyserver/ --include='*.py' | grep -c 'for.*range(10)'
```

**Accept when:**
- All test server implementations in dummyserver/ use socket.bind((host, port)) with explicit parameters
- At least one implementation demonstrates dual-stack binding by iterating through socket.getaddrinfo results with AF_UNSPEC or AF_INET6
- Retry logic with errno.EADDRINUSE handling is present in socket binding code with at least 10 retry attempts
- Socket options are configured with sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1) and sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) before binding
- Allocated port is extracted using sock.getsockname()[1] and reused for subsequent address family bindings

<enforcement>
Claude Code MUST NOT skip or defer verification of socket binding implementations. All test server socket creation MUST follow dual-stack binding patterns with port reuse and retry logic as specified in R-SOCKET-001.
</enforcement>
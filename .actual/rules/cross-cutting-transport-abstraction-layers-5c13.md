# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Transport Abstraction Layers

These rules are ALWAYS ACTIVE for all HTTP/1.1 and HTTP/2 connection implementations in `src/urllib3/`, SSL/TLS transport wrappers (SSLTransport, pyOpenSSL adapters), platform-specific implementations (Emscripten fetch, native sockets), and test infrastructure socket handlers in `test/` and `dummyserver/`.

### Rules

- **R-TRANSPORT-001** SHOULD: Transport abstraction layers (SSLTransport, HTTP2Connection, pyOpenSSL wrappers) SHOULD delegate to underlying socket send() rather than implementing custom buffering.
- **R-TRANSPORT-002** MUST: All string data MUST be encoded to bytes before send()—use `.encode('utf-8')` for body content and `.encode('ascii')` for HTTP headers per RFC 7230.
- **R-TRANSPORT-003** MUST: Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets.
- **R-TRANSPORT-004** MUST: No direct socket write operations MUST bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send).
- **R-TRANSPORT-005** SHOULD: In test infrastructure, construct HTTP responses with explicit `\r\n` line endings and Content-Length headers to ensure protocol compliance.
- **R-TRANSPORT-006** MUST: For transport abstraction layers (SSLTransport, HTTP2Connection), delegate to underlying socket.send() rather than buffering—preserve message boundaries and ordering.

### Verify

```bash
# Count socket.send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# Find all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run transport and socket-level tests
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- All string data is encoded to bytes before transmission
- Error handling for partial sends and socket errors is present in transport layers

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport implementations MUST be audited for compliance with R-TRANSPORT-001 through R-TRANSPORT-006. CI build failure is mandatory if transport implementations bypass send() primitive.
</enforcement>
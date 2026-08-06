# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Http Response Construction

These rules are ALWAYS ACTIVE for all HTTP client transport implementations, SSL/TLS wrappers, platform-specific adapters, and test infrastructure socket handlers in urllib3.

### Rules

- **R-HTTP-SEND-001** MUST: HTTP response construction in test infrastructure MUST use sock.send() with complete HTTP/1.1 formatted responses including status line, headers, and body.
- **R-HTTP-SEND-002** MUST: All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) MUST use socket.send() or connection.send() as the terminal transmission operation for message delivery.
- **R-HTTP-SEND-003** MUST: No direct socket write operations outside the send() primitive are permitted in public API methods (connect, putrequest, putheader, endheaders, send).
- **R-HTTP-SEND-004** MUST: All string data MUST be encoded to bytes before send()—use .encode('utf-8') for body content and .encode('ascii') for HTTP headers per RFC 7230.
- **R-HTTP-SEND-005** MUST: Test infrastructure socket handlers MUST construct HTTP responses with explicit \r\n line endings and Content-Length headers to ensure protocol compliance.
- **R-HTTP-SEND-006** SHOULD: Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets.
- **R-HTTP-SEND-007** SHOULD: For transport abstraction layers (SSLTransport, HTTP2Connection), delegate to underlying socket.send() rather than buffering to preserve message boundaries and ordering.

### Verify

```bash
# Count send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# Find all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run transport and socket-level test suites
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- All string data is encoded to bytes before transmission (UTF-8 for body, ASCII for headers)
- HTTP responses in test infrastructure include explicit \r\n line endings and Content-Length headers

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All transport layer changes and new implementations MUST be validated against the verify commands and acceptance criteria. Violations result in CI build failure and code review rejection.
</enforcement>
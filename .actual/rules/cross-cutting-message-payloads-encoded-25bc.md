# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Message Payloads Encoded

These rules are ALWAYS ACTIVE for all HTTP client transport implementations, SSL/TLS wrappers, platform-specific adapters, and test infrastructure socket handlers in the urllib3 codebase.

### Rules

- **R-SEND-001** MUST: Message payloads MUST be encoded to bytes (via .encode('utf-8') or equivalent) before invoking send().
- **R-SEND-002** MUST: All transport implementations (HTTP/1.1, HTTP/2, pyOpenSSL, SSLTransport, Emscripten) MUST use socket.send() or connection.send() as the terminal transmission operation for HTTP messages.
- **R-SEND-003** MUST: No direct socket write operations outside the send() primitive contract are permitted in public API methods (connect, putrequest, putheader, endheaders, send).
- **R-SEND-004** SHOULD: Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets.
- **R-SEND-005** SHOULD: Use .encode('utf-8') for body content and .encode('ascii') for HTTP headers per RFC 7230.
- **R-SEND-006** SHOULD: In test infrastructure, construct HTTP responses with explicit \r\n line endings and Content-Length headers to ensure protocol compliance.

### Verify

```bash
# Count send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# List all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run transport layer test suite
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- All string payloads are encoded to bytes before send() invocation
- Integration test suite (test/test_ssltransport.py, test/with_dummyserver/test_socketlevel.py) validates socket-level message transmission

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport layer changes require validation that send() is the terminal transmission primitive and all payloads are byte-encoded before transmission.
</enforcement>
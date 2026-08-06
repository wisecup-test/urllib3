# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Public Methods Connect

These rules are ALWAYS ACTIVE for all HTTP client transport implementations, connection classes, and test infrastructure in urllib3 that handle socket-level message transmission across HTTP/1.1, HTTP/2, SSL/TLS, and platform-specific transports.

### Rules

- **R-SEND-001** MUST NOT: Public API methods (connect, putrequest, putheader, endheaders, send) MUST NOT bypass the send() primitive for message transmission. All HTTP message bytes destined for the network MUST flow through socket.send() or connection.send() as the terminal operation.

- **R-SEND-002** MUST: All transport implementations (native sockets, pyOpenSSL adapters, SSLTransport, HTTP/2 connections, Emscripten fetch) MUST delegate message transmission to the underlying send() primitive rather than implementing alternative write paths.

- **R-SEND-003** MUST: String data MUST be encoded to bytes before send()—use .encode('utf-8') for body content and .encode('ascii') for HTTP headers per RFC 7230.

- **R-SEND-004** SHOULD: Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets to handle partial sends.

- **R-SEND-005** SHOULD: Test infrastructure socket handlers MUST construct HTTP responses with explicit \r\n line endings and Content-Length headers to ensure protocol compliance when injecting responses via sock.send().

### Verify

```bash
# Count send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# List all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run socket-level transmission tests
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v

# Verify no direct socket write operations bypass send() in public API methods
grep -r 'def \(connect\|putrequest\|putheader\|endheaders\|send\)' src/urllib3/ | grep -v 'send()' | head -20
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- Integration test suite (test/test_ssltransport.py, test/with_dummyserver/test_socketlevel.py) validates socket-level message transmission across protocol variants
- All string data is encoded to bytes before send() with appropriate charset (utf-8 for body, ascii for headers)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All transport layer changes MUST be validated against R-SEND-001 through R-SEND-005. CI build failure is mandatory if transport implementations bypass send() primitive. Code review rejection is required for direct socket write operations outside send() contract.
</enforcement>
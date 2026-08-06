# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Http Message Transmission

These rules are ALWAYS ACTIVE for all HTTP client transport implementations, connection classes, and test infrastructure that handle socket-level message transmission across public API boundaries.

### Rules

- **R-HTTP-TX-001** MUST: All HTTP message transmission at public API boundaries MUST use socket.send() or connection.send() as the terminal operation for byte sequence delivery.
- **R-HTTP-TX-002** MUST: All transport implementations (HTTP/1.1, HTTP/2, pyOpenSSL, SSLTransport, Emscripten) MUST delegate to underlying socket.send() rather than buffering—preserve message boundaries and ordering.
- **R-HTTP-TX-003** MUST: Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets.
- **R-HTTP-TX-004** MUST: Ensure all string data is encoded to bytes before send()—use .encode('utf-8') for body content and .encode('ascii') for HTTP headers per RFC 7230.
- **R-HTTP-TX-005** SHOULD: In test infrastructure, construct HTTP responses with explicit \r\n line endings and Content-Length headers to ensure protocol compliance.
- **R-HTTP-TX-006** SHOULD: Centralize encoding logic in message construction utilities; enforce encoding validation in code review and linting.

### Verify

```bash
# Count send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# Find all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run transport layer tests
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- All string data is encoded to bytes before transmission (UTF-8 for body, ASCII for headers)
- Error handling for partial sends and socket errors is present in transport layers

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport implementations MUST be audited for send() usage compliance. Integration tests MUST pass before accepting changes to socket-level transmission logic.
</enforcement>
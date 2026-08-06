# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Test Socket Handlers

These rules are ALWAYS ACTIVE for all HTTP client transport implementations, SSL/TLS wrappers, platform-specific adapters, and test infrastructure socket handlers in src/urllib3/ and test/ directories.

### Rules

- **R-SOCK-001** SHOULD: Test socket handlers SHOULD construct responses using byte string literals (b"...") for protocol-level testing to ensure exact wire format.

### Verify

```bash
# Count socket.send() usage across transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# Verify send() method definitions in transport layer
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run socket-level transmission tests
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- Test socket handlers use byte string literals (b"...") for HTTP response construction with explicit \r\n line endings and Content-Length headers

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport implementations and test socket handlers MUST conform to the send() primitive contract. Violations detected by grep patterns or test failures MUST block acceptance.
</enforcement>

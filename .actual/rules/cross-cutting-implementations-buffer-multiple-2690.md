# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Implementations Buffer Multiple

These rules are ALWAYS ACTIVE for all HTTP/1.1 and HTTP/2 connection implementations, SSL/TLS transport wrappers, platform-specific implementations, and test infrastructure socket handlers in the urllib3 codebase.

### Rules

- **R-SEND-001** MAY: Implementations MAY buffer multiple send() calls internally but MUST preserve message ordering and boundary semantics.

### Verify

```bash
# Count send() usage in transport implementations
grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l

# Find all send() method definitions
grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'

# Run socket-level and transport tests
python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v
```

**Accept when:**
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)
- Message ordering is preserved across all buffered send() calls
- Byte-level message boundaries are maintained in all transport implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport implementations must be audited to confirm send() is the terminal transmission primitive. Integration tests must pass to validate socket-level message transmission semantics.
</enforcement>

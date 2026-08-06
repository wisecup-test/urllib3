# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Transport Abstractions Implement

These rules are ALWAYS ACTIVE for all HTTP transport implementations in `src/urllib3/` (connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), socket-level test infrastructure in `test/with_dummyserver/test_socketlevel.py`, `test/test_ssltransport.py`, `test/test_http2_connection.py`, and test harness socket handlers in `dummyserver/testcase.py`.

### Rules

- **R-TRANSPORT-001** SHOULD: Transport abstractions SHOULD implement common interface contracts (send, recv, close, fileno) to enable polymorphic usage across socket, SSL, and custom transports.
- **R-TRANSPORT-002** MUST: All transport implementations expose a send() method accepting bytes, bytearray, or memoryview.
- **R-TRANSPORT-003** MUST: Send implementations MUST handle partial writes with a loop and use byte_view slicing to track progress.
- **R-TRANSPORT-004** MUST: String inputs to send() MUST be converted to UTF-8 bytes explicitly before transmission.
- **R-TRANSPORT-005** MUST: Send implementations MUST propagate socket errors appropriately to callers.
- **R-TRANSPORT-006** SHOULD: HTTP/2 send operations SHOULD be coordinated with stream multiplexing using threading primitives to respect frame boundaries.
- **R-TRANSPORT-007** SHOULD: Platform adapters (Emscripten, PyOpenSSL) SHOULD document semantic differences between native send() and adapted APIs in module docstrings.
- **R-TRANSPORT-008** MUST: New transport implementations MUST be validated with dummyserver/testcase.py infrastructure to confirm protocol compliance.

### Verify

```bash
# Verify send method presence in transport implementations
grep -r 'def send(' src/urllib3/ --include='*.py' | grep -E '(connection|transport|ssl)'

# Verify test infrastructure uses socket-level send operations
grep -r '\.send\(' test/ --include='*.py' | grep -E '(sock|conn|ssock)'

# Validate socket-level integration tests pass
python -m pytest test/test_socketlevel.py test/test_ssltransport.py test/test_http2_connection.py -v
```

**Accept when:**
- All transport implementations in `src/urllib3/` expose a send() method accepting bytes, verified by grep showing consistent method signatures across connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, and contrib/emscripten/fetch.py.
- Integration tests (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py) pass, confirming send operations work correctly across socket, SSL, and HTTP/2 transports.
- Code review confirms new transport implementations handle partial writes, maintain interface contracts (send, recv, close, fileno), and convert strings to UTF-8 bytes explicitly.
- Socket-level tests with dummyserver infrastructure validate protocol compliance and proper header/body separation.

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block acceptance until send implementation is corrected. Code review MUST identify missing send() methods or incorrect signatures before approval. Runtime errors from incorrect send usage trigger bug reports and regression test creation.
</enforcement>
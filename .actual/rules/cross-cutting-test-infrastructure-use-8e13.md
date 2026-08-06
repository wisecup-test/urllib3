# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Test Infrastructure Use

These rules are ALWAYS ACTIVE for all HTTP transport implementations in `src/urllib3/` (connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), socket-level test infrastructure in `test/with_dummyserver/test_socketlevel.py`, `test/test_ssltransport.py`, `test/test_http2_connection.py`, and test harness socket handlers in `dummyserver/testcase.py`.

### Rules

- **R-SOCKET-001** SHOULD: Test infrastructure SHOULD use socket-level send operations (sock.send()) to construct protocol-compliant responses for integration testing.
- **R-SOCKET-002** MUST: All transport implementations in src/urllib3/ MUST expose a send() method accepting bytes, bytearray, or memoryview.
- **R-SOCKET-003** MUST: Send implementations MUST handle partial writes with a loop and use byte_view slicing to track progress.
- **R-SOCKET-004** MUST: String data MUST be converted to UTF-8 bytes explicitly before transmission via send().
- **R-SOCKET-005** MUST: Socket errors MUST be propagated appropriately from send() implementations.
- **R-SOCKET-006** SHOULD: HTTP/2 send operations SHOULD be centralized in http2/connection.py with explicit threading coordination to prevent frame interleaving corruption.
- **R-SOCKET-007** SHOULD: Platform adapters (Emscripten, PyOpenSSL) SHOULD document semantic differences between native send() and adapted APIs in module docstrings.
- **R-SOCKET-008** MAY: Test mock implementations MAY simulate send failures or partial writes for error path validation, provided deviations are documented in test comments.

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
- All transport implementations in src/urllib3/ expose send() method accepting bytes, verified by grep showing consistent method signatures across connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, and contrib/emscripten/fetch.py
- Integration tests (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py) pass, confirming send operations work correctly across socket, SSL, and HTTP/2 transports
- Code review confirms new transport implementations handle partial writes and maintain interface contracts (send, recv, close, fileno)
- All string-to-bytes conversions use explicit .encode('utf-8') pattern
- HTTP/2 implementations coordinate send operations with threading primitives and respect frame boundaries
- Platform adapters document semantic differences in module docstrings

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block acceptance. Code review MUST verify send() method signatures, partial write handling, and encoding semantics for all new transport implementations. Runtime errors from incorrect send usage trigger bug reports and regression test creation.
</enforcement>
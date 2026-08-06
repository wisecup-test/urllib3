# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Http Transport Implementations

These rules are ALWAYS ACTIVE for all HTTP transport implementations in `src/urllib3/` (connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), socket-level test infrastructure in `test/with_dummyserver/test_socketlevel.py`, `test/test_ssltransport.py`, `test/test_http2_connection.py`, and test harness socket handlers in `dummyserver/testcase.py`.

### Rules

- **R-HTTP-TRANSPORT-001** MUST: All HTTP transport implementations MUST expose a `send()` method accepting bytes or byte-like objects (bytes, bytearray, memoryview) for transmitting data over the underlying connection.

- **R-HTTP-TRANSPORT-002** MUST: Transport implementations MUST handle partial writes by looping until all bytes are transmitted, using byte_view slicing to track progress (reference: util/ssltransport.py pattern).

- **R-HTTP-TRANSPORT-003** MUST: String data MUST be explicitly converted to UTF-8 bytes before passing to send() operations.

- **R-HTTP-TRANSPORT-004** MUST: Socket-level errors MUST be propagated appropriately without silent suppression or generic wrapping that obscures the underlying transport failure.

- **R-HTTP-TRANSPORT-005** SHOULD: New transport implementations SHOULD document semantic differences between native send() and adapted APIs (e.g., js_xhr.send() for Emscripten, OpenSSL.SSL.Connection.send() for PyOpenSSL) in module docstrings.

- **R-HTTP-TRANSPORT-006** SHOULD: HTTP/2 implementations SHOULD coordinate send operations with stream multiplexing using threading primitives to prevent frame interleaving corruption.

- **R-HTTP-TRANSPORT-007** MAY: Test mock implementations MAY simulate send failures or partial writes for error path validation, provided deviations are documented in test comments and scope is limited to test modules.

### Verify

```bash
# Verify send method presence in transport implementations
grep -r 'def send(' src/urllib3/ --include='*.py' | grep -E '(connection|transport|ssl)'

# Verify test infrastructure uses socket-level send operations
grep -r '\.send\(' test/ --include='*.py' | grep -E '(sock|conn|ssock)'

# Validate socket-level integration tests pass
python -m pytest test/test_socketlevel.py test/test_ssltransport.py test/test_http2_connection.py -v

# Verify send implementations handle partial writes
grep -r 'while.*send\|for.*send' src/urllib3/ --include='*.py' -A 2

# Verify UTF-8 encoding is explicit in send operations
grep -r '\.encode.*utf-8\|\.encode()' test/ --include='*.py' | grep -E '(send|socket)'
```

**Accept when:**
- All transport implementations in `src/urllib3/` expose a `send()` method accepting bytes, verified by grep showing consistent method signatures across connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, and contrib/emscripten/fetch.py
- Integration tests (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py) pass, confirming send operations work correctly across socket, SSL, and HTTP/2 transports
- Code review confirms new transport implementations handle partial writes with loop-based byte tracking and maintain interface contracts (send, recv, close, fileno)
- String-to-bytes conversion is explicit (e.g., `.encode('utf-8')`) in all send call sites
- Socket errors are propagated without silent suppression
- Platform-specific adapters document semantic differences in module docstrings

<enforcement>
Claude Code MUST NOT skip or defer verification. All transport implementations MUST expose send() accepting bytes/byte-like objects. Integration tests MUST pass. Code review MUST confirm partial write handling and error propagation. Violations block merge until corrected.
</enforcement>
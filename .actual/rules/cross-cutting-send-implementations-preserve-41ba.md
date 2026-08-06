# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Send Implementations Preserve

These rules are ALWAYS ACTIVE for all HTTP transport implementations in src/urllib3/ (connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), socket-level test infrastructure in test/with_dummyserver/test_socketlevel.py, test/test_ssltransport.py, test/test_http2_connection.py, and test harness socket handlers in dummyserver/testcase.py.

### Rules

- **R-SEND-001** MUST: Send implementations MUST preserve byte encoding semantics, converting string data to UTF-8 bytes when necessary before transmission.

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
- All transport implementations in src/urllib3/ expose send() method accepting bytes, verified by grep showing consistent method signatures
- Integration tests (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py) pass, confirming send operations work correctly across socket, SSL, and HTTP/2 transports
- Code review confirms new transport implementations handle partial writes and maintain interface contracts (send, recv, close, fileno)
- Send implementations explicitly convert strings to UTF-8 bytes before transmission (verified by code inspection or test coverage)

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block merge until send implementation is corrected. Code review MUST identify missing send() methods or incorrect signatures. Runtime errors from incorrect send usage trigger bug reports and regression test creation.
</enforcement>
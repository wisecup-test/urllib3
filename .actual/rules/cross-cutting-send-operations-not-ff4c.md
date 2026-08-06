# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Send Operations Not

These rules are ALWAYS ACTIVE for all HTTP transport implementations, socket-level test infrastructure, and test harness code that perform byte-level data transmission across heterogeneous transport layers (native sockets, SSL/TLS wrappers, HTTP/2 multiplexing, browser environments).

### Rules

- **R-SEND-001** MUST NOT: Send operations MUST NOT assume complete transmission in a single call; implementations MUST NOT ignore return values indicating partial writes.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block merge until send implementation is corrected. Code review identifies missing send() methods or incorrect signatures and requires revision before approval. Runtime errors from incorrect send usage trigger bug reports and regression test creation.
</enforcement>
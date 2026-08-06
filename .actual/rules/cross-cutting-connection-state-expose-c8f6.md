# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Connection State Expose

These rules are ALWAYS ACTIVE for all Emscripten/WebAssembly HTTP integration code in `src/urllib3/contrib/emscripten/` and related browser-based Python runtime environments where urllib3 must bridge synchronous Python interfaces with asynchronous JavaScript Fetch API primitives.

### Rules

- **R-EMSCRIPTEN-001** MUST: Connection state MUST expose `is_closed`, `closed`, `close`, `readable`, and `writable` properties to satisfy urllib3's public API contracts.
- **R-EMSCRIPTEN-002** MUST: Use message queue send operations (`_fetcher.send` or `js_xhr.send`) for all cross-runtime communication between Python and JavaScript execution contexts.
- **R-EMSCRIPTEN-003** MUST: Bind `AbortController.abort` early in the request lifecycle to ensure cancellation handlers are available before fetch initiation.
- **R-EMSCRIPTEN-004** MUST: Implement `wait_for_streaming_ready` with appropriate timeout handling to prevent indefinite blocking on failed requests.
- **R-EMSCRIPTEN-005** SHOULD: Use `to_js()` for all data crossing the Python-JavaScript boundary to ensure proper type conversion and memory management.
- **R-EMSCRIPTEN-006** SHOULD: Implement streaming and chunked transfer where possible to mitigate message queue serialization overhead for large request/response bodies.

### Verify

```bash
# Verify message queue send operations are present
grep -r '_fetcher\.send\|js_xhr\.send' src/urllib3/contrib/emscripten/ | grep -q 'send'

# Verify AbortController binding is implemented
grep -r 'js_abort_controller\.abort\.bind' src/urllib3/contrib/emscripten/ | grep -q 'abort'

# Verify all five API contract properties are exposed
grep -r 'is_closed\|closed\|close\|readable\|writable' src/urllib3/contrib/emscripten/ | wc -l | awk '$1 >= 5'
```

**Accept when:**
- Message queue send operations (`_fetcher.send` or `js_xhr.send`) are present in Emscripten fetch implementation
- AbortController binding is implemented for request cancellation
- All five API contract properties (`is_closed`, `closed`, `close`, `readable`, `writable`) are exposed in the connection interface
- Integration tests pass in actual Emscripten/browser environments

<enforcement>
Claude Code MUST NOT skip or defer verification. All five rules must be satisfied before accepting changes to Emscripten HTTP integration code. Violations detected by CI pipeline grep-based verification, unit tests validating AbortController integration, and integration tests in browser environments will cause build failure.
</enforcement>
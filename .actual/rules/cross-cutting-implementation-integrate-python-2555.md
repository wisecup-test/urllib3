# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Implementation Integrate Python

These rules are ALWAYS ACTIVE for all Python code in `src/urllib3/contrib/emscripten/` that implements HTTP request handling through JavaScript Fetch API and AbortController integration in WebAssembly/Emscripten environments.

### Rules

- **R-EMSCRIPTEN-001** MUST: The implementation MUST integrate with Python's io, json, and email.parser core libraries for request/response serialization.
- **R-EMSCRIPTEN-002** MUST: Use to_js() for all data crossing the Python-JavaScript boundary to ensure proper type conversion and memory management.
- **R-EMSCRIPTEN-003** MUST: Bind AbortController.abort early in request lifecycle to ensure cancellation handlers are available before fetch initiation.
- **R-EMSCRIPTEN-004** MUST: Implement wait_for_streaming_ready with appropriate timeout handling to prevent indefinite blocking on failed requests.
- **R-EMSCRIPTEN-005** MUST: Maintain API contract compatibility by implementing all required properties (is_closed, closed, close, readable, writable) even if some are no-ops in the JavaScript context.
- **R-EMSCRIPTEN-006** MUST: Implement message queue send operations (_fetcher.send or js_xhr.send) for cross-runtime communication between Python and JavaScript execution contexts.
- **R-EMSCRIPTEN-007** MUST: Expose AbortController binding for request cancellation to support timeout and user-initiated request cancellation semantics.

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
- Message queue send operations (_fetcher.send or js_xhr.send) are present in Emscripten fetch implementation
- AbortController binding is implemented for request cancellation
- All five API contract properties (is_closed, closed, close, readable, writable) are exposed in the connection interface
- Integration with Python's io, json, and email.parser libraries is demonstrated in request/response handling code
- to_js() conversion is used for all Python-JavaScript boundary data transfers
- wait_for_streaming_ready includes timeout handling logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST pass before accepting changes to Emscripten HTTP implementation. CI pipeline MUST fail if message queue boundaries or AbortController patterns are removed. Code review MUST flag any changes that bypass JavaScript API integration.
</enforcement>
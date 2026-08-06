# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Streaming Response Handling

These rules are ALWAYS ACTIVE for all files in `src/urllib3/contrib/emscripten/` that implement HTTP request handling through JavaScript Fetch API integration in WebAssembly/Emscripten environments.

### Rules

- **R-EMSCRIPTEN-001** SHOULD: Streaming response handling SHOULD use wait_for_streaming_ready to coordinate asynchronous fetch completion with synchronous Python code expectations.
- **R-EMSCRIPTEN-002** MUST: Use to_js() for all data crossing the Python-JavaScript boundary to ensure proper type conversion and memory management.
- **R-EMSCRIPTEN-003** MUST: Bind AbortController.abort early in request lifecycle to ensure cancellation handlers are available before fetch initiation.
- **R-EMSCRIPTEN-004** MUST: Implement wait_for_streaming_ready with appropriate timeout handling to prevent indefinite blocking on failed requests.
- **R-EMSCRIPTEN-005** MUST: Maintain API contract compatibility by implementing all required properties (is_closed, closed, close, readable, writable) even if some are no-ops in the JavaScript context.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to Emscripten HTTP integration. CI build MUST fail if message queue boundaries or AbortController patterns are removed. Code review MUST flag any changes that bypass JavaScript API integration.
</enforcement>
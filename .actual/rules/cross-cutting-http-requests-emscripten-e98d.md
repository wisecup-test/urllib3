# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Http Requests Emscripten

These rules are ALWAYS ACTIVE for all HTTP request implementations in Emscripten/WebAssembly environments, particularly in `src/urllib3/contrib/emscripten/` and related cross-runtime communication layers.

### Rules

- **R-EMSCRIPTEN-001** MUST: HTTP requests in Emscripten environments MUST use JavaScript Fetch API or XMLHttpRequest as the transport layer, invoked via message queue boundaries (_fetcher.send, js_xhr.send).
- **R-EMSCRIPTEN-002** MUST: All data crossing the Python-JavaScript boundary MUST use to_js() for proper type conversion and memory management.
- **R-EMSCRIPTEN-003** MUST: AbortController.abort MUST be bound early in the request lifecycle to ensure cancellation handlers are available before fetch initiation.
- **R-EMSCRIPTEN-004** MUST: The API contract surface (is_closed, closed, close, readable, writable) MUST be implemented to maintain urllib3's HTTPConnection interface compatibility.
- **R-EMSCRIPTEN-005** SHOULD: wait_for_streaming_ready SHOULD be implemented with appropriate timeout handling to prevent indefinite blocking on failed requests.

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
- to_js() is used for all cross-boundary data conversions
- wait_for_streaming_ready includes timeout handling logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All five verification conditions MUST pass before accepting changes to Emscripten HTTP request implementations. Violations detected in CI pipeline MUST cause build failure. Code review MUST flag any changes that bypass JavaScript API integration or remove message queue boundaries.
</enforcement>
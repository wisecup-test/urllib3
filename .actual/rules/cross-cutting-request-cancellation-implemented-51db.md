# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Request Cancellation Implemented

These rules are ALWAYS ACTIVE for all files in `src/urllib3/contrib/emscripten/` and related WebAssembly/Emscripten HTTP transport implementations.

### Rules

- **R-EMSCRIPTEN-001** MUST: Request cancellation MUST be implemented using JavaScript AbortController bound to abort handlers (js_abort_controller.abort.bind).

### Verify

```bash
# Verify message queue send operations are present
grep -r '_fetcher\.send\|js_xhr\.send' src/urllib3/contrib/emscripten/ | grep -q 'send'

# Verify AbortController binding is implemented for request cancellation
grep -r 'js_abort_controller\.abort\.bind' src/urllib3/contrib/emscripten/ | grep -q 'abort'

# Verify all five API contract properties are exposed
grep -r 'is_closed\|closed\|close\|readable\|writable' src/urllib3/contrib/emscripten/ | wc -l | awk '$1 >= 5'
```

**Accept when:**
- Message queue send operations (_fetcher.send or js_xhr.send) are present in Emscripten fetch implementation
- AbortController binding is implemented for request cancellation
- All five API contract properties (is_closed, closed, close, readable, writable) are exposed in the connection interface

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to Emscripten HTTP transport implementations.
</enforcement>
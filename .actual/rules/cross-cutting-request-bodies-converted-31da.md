# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Request Bodies Converted

These rules are ALWAYS ACTIVE for all code in `src/urllib3/contrib/emscripten/` that implements HTTP request handling through JavaScript Fetch API and XMLHttpRequest in WebAssembly environments.

### Rules

- **R-EMSCRIPTEN-001** MUST: Request bodies MUST be converted to JavaScript-compatible representations using to_js() before transmission across the Python-JavaScript boundary.

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
- Request body conversion using to_js() is applied before all cross-boundary transmissions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to Emscripten HTTP request handling. Violations in CI pipeline must fail the build.
</enforcement>
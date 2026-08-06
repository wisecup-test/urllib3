# Adopt JavaScript Fetch API with Abort Controller for Async HTTP in Emscripten Environment: Request Bodies Converted

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Context

- The urllib3 library requires HTTP request capabilities when running in WebAssembly/Emscripten environments where traditional socket-based networking is unavailable
- JavaScript Fetch API and XMLHttpRequest provide the only available HTTP transport mechanisms in browser-based Python runtimes compiled to WebAssembly
- Abort controllers enable cancellation semantics for in-flight HTTP requests, matching the timeout and cancellation patterns expected by urllib3's API contract
- The Emscripten contrib module bridges Python's synchronous HTTP interface expectations with JavaScript's asynchronous fetch primitives through message passing

## Problem Statement

When urllib3 runs in Emscripten/WebAssembly environments, it cannot use standard Python socket libraries for HTTP communication. The system must provide HTTP request capabilities that integrate with JavaScript runtime APIs while maintaining urllib3's expected interface contracts for connection lifecycle, request/response handling, and cancellation semantics.

## Decision

1. MUST: Request bodies MUST be converted to JavaScript-compatible representations using to_js() before transmission across the Python-JavaScript boundary

## Policy Block

- MUST Request bodies MUST be converted to JavaScript-compatible representations using to_js() before transmission across the Python-JavaScript boundary

## Rationale

- Evidence shows explicit message queue boundaries (_fetcher.send, js_xhr.send) indicating cross-runtime communication between Python and JavaScript execution contexts
- The presence of js_abort_controller.abort.bind demonstrates integration with JavaScript's standard cancellation mechanism, required for timeout and abort semantics
- Detection of wait_for_streaming_ready indicates a concurrency coordination pattern bridging JavaScript's Promise-based async model with Python's synchronous execution model
- The API contract surface (is_closed, closed, close, readable, writable) matches urllib3's HTTPConnection interface requirements, ensuring drop-in compatibility

## Consequences

Positive:
- Enables urllib3 to function in WebAssembly/browser environments where native sockets are unavailable, expanding platform compatibility
- Leverages browser-native HTTP implementations (Fetch API, XHR) which handle CORS, security policies, and protocol details automatically
- Provides proper cancellation semantics through AbortController, allowing timeouts and user-initiated request cancellation
- Maintains urllib3's existing API surface, minimizing code changes required in client applications

Negative:
- Introduces message queue overhead and serialization costs at the Python-JavaScript boundary for every request/response
- Creates dependency on JavaScript runtime availability and specific browser APIs, limiting portability to pure Python environments
- Adds complexity through async-to-sync bridging (wait_for_streaming_ready), potentially impacting performance and debuggability
- Requires maintenance of platform-specific code paths and testing infrastructure for Emscripten environments

## Alternatives

- Implement a pure Python HTTP stack using Emscripten's socket emulation layer (rejected)
  Rejected because: Emscripten's socket emulation is incomplete and does not support the full range of socket operations required by urllib3's connection pooling and keep-alive mechanisms
  When valid: If Emscripten gains full POSIX socket API compatibility with WebSocket-based transport
- Use pyodide's fetch API wrapper directly instead of custom integration (rejected)
  Rejected because: Pyodide's fetch wrapper does not provide the connection lifecycle management and cancellation semantics required by urllib3's architecture
  When valid: For simpler HTTP client libraries without connection pooling requirements
- Fork urllib3 into a separate emscripten-specific package (rejected)
  Rejected because: Creates maintenance burden and fragments the ecosystem; contrib module approach allows single codebase with platform-specific adapters
  When valid: If the architectural differences become too significant to maintain in a single codebase

## Risks

- JavaScript Fetch API limitations (e.g., no streaming upload support in some browsers) may prevent certain urllib3 features from working
  Mitigation: Document browser compatibility requirements and feature limitations; provide fallback to XHR where Fetch API is insufficient
  Owner: urllib3 Emscripten maintainers
- Message queue serialization overhead may cause performance degradation for large request/response bodies
  Mitigation: Implement streaming and chunked transfer where possible; benchmark and document performance characteristics
  Owner: engineering team
- Async-to-sync bridging may cause deadlocks or event loop starvation in complex concurrent scenarios
  Mitigation: Thoroughly test concurrent request patterns; document threading and event loop constraints for Emscripten environments
  Owner: engineering team

## Implementation Notes

- Use to_js() for all data crossing the Python-JavaScript boundary to ensure proper type conversion and memory management
- Bind AbortController.abort early in request lifecycle to ensure cancellation handlers are available before fetch initiation
- Implement wait_for_streaming_ready with appropriate timeout handling to prevent indefinite blocking on failed requests
- Maintain API contract compatibility by implementing all required properties (is_closed, closed, close, readable, writable) even if some are no-ops in the JavaScript context

## Continuation Context


Verify commands:
- grep -r '_fetcher\.send\|js_xhr\.send' src/urllib3/contrib/emscripten/ | grep -q 'send'
- grep -r 'js_abort_controller\.abort\.bind' src/urllib3/contrib/emscripten/ | grep -q 'abort'
- grep -r 'is_closed\|closed\|close\|readable\|writable' src/urllib3/contrib/emscripten/ | wc -l | awk '$1 >= 5'

Accept when:
- Message queue send operations (_fetcher.send or js_xhr.send) are present in Emscripten fetch implementation
- AbortController binding is implemented for request cancellation
- All five API contract properties (is_closed, closed, close, readable, writable) are exposed in the connection interface

## Enforcement

- Verified by: Automated grep-based verification in CI pipeline checking for required message queue patterns
- Verified by: Unit tests validating AbortController integration and cancellation behavior
- Verified by: Integration tests running in actual Emscripten/browser environments
- Violation handling: CI build fails if message queue boundaries or AbortController patterns are removed
- Violation handling: Code review flags any changes to Emscripten fetch implementation that bypass JavaScript API integration
- Violation handling: Runtime errors in browser environments when API contracts are not satisfied
- Exception process: Document alternative implementation approach in ADR amendment with rationale
- Exception process: Obtain approval from urllib3 maintainers for architectural changes to Emscripten integration
- Exception process: Ensure backward compatibility or provide migration path for existing Emscripten users
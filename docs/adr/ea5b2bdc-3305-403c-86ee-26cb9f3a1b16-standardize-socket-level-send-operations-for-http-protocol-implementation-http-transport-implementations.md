# Standardize Socket-Level Send Operations for HTTP Protocol Implementation: Http Transport Implementations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements HTTP/1.1, HTTP/2, and alternative transport protocols (Emscripten, PyOpenSSL) requiring consistent low-level socket communication patterns
- Multiple integration points exist across test infrastructure (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py), production code (http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), and test harness (dummyserver/testcase.py)
- Core Python libraries (socket, ssl, io, typing) are detected alongside protocol-specific implementations requiring uniform send/recv abstractions
- The pattern spans 8 files with 92.73% confidence, indicating systematic architectural consistency rather than isolated implementation choices
- Socket-level operations must support multiple transport layers (SSL/TLS via ssl and OpenSSL.SSL, HTTP/2 via custom connection handling, browser fetch via js bindings) while maintaining common interface contracts

## Problem Statement

HTTP client libraries must abstract socket-level send operations across heterogeneous transport implementations (native sockets, SSL/TLS wrappers, HTTP/2 multiplexing, browser environments) while maintaining testability, protocol compliance, and consistent error handling. Without standardized send patterns, each transport layer risks divergent behavior in buffering, encoding, error propagation, and connection lifecycle management.

## Decision

1. MUST: All HTTP transport implementations MUST expose a send() method accepting bytes or byte-like objects for transmitting data over the underlying connection

## Policy Block

- MUST All HTTP transport implementations MUST expose a send() method accepting bytes or byte-like objects for transmitting data over the underlying connection

In scope:
- All HTTP transport implementations in src/urllib3/ (connection.py, http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py)
- Socket-level test infrastructure in test/with_dummyserver/test_socketlevel.py, test/test_ssltransport.py, test/test_http2_connection.py
- Test harness socket handlers in dummyserver/testcase.py
- Any new transport adapters or protocol implementations requiring byte-level data transmission

Out of scope:
- High-level HTTP request APIs that operate on request/response objects rather than raw bytes
- Connection pooling and retry logic (operates above transport layer)
- URL parsing, header encoding, and request serialization (precedes send operations)
- Response parsing and body decoding (follows recv operations)

Exceptions:
- EXC-001: Browser-based environments (Emscripten) where native socket APIs are unavailable and JavaScript fetch/XHR APIs provide the only transmission mechanism
- EXC-002: Test mock implementations that simulate send failures or partial writes for error path validation

## Rationale

- Evidence shows systematic use of send() operations across 8 files spanning production transports (http2/connection.py, util/ssltransport.py, contrib/pyopenssl.py, contrib/emscripten/fetch.py), test infrastructure (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py), and test harness (dummyserver/testcase.py), indicating architectural pattern rather than isolated implementation
- Core library detection (socket, ssl, io, typing, threading) combined with boundaries.message_queues evidence (sock.send(), self.send(), connection.send()) demonstrates consistent abstraction layer for byte transmission across heterogeneous transports
- Public API contracts (send, recv, close, fileno, sendall) appear in multiple contexts (util/ssltransport.py, contrib/pyopenssl.py, test_ssltransport.py), confirming intentional interface standardization for transport polymorphism
- HTTP protocol compliance requires precise control over byte transmission timing and encoding (evidenced by explicit UTF-8 encoding in test_socketlevel.py and header/body separation in HTTP/1.1 responses), necessitating low-level send abstractions rather than higher-level stream APIs

## Consequences

Positive:
- Transport layer polymorphism enables testing with mock sockets, production use with SSL/TLS, HTTP/2 multiplexing, and platform-specific adapters (PyOpenSSL, Emscripten) through common interface
- Socket-level control allows precise HTTP protocol implementation including header/body separation, chunked encoding, and connection lifecycle management
- Consistent send semantics across implementations reduce integration bugs when switching transports or testing edge cases
- Test infrastructure can construct protocol-compliant responses using same send operations as production code, improving test fidelity

Negative:
- Low-level send abstractions expose complexity of partial writes, encoding, and error handling to all transport implementations rather than centralizing in single location
- Platform-specific adapters (Emscripten, PyOpenSSL) must bridge impedance mismatch between byte-oriented send() interface and native APIs (JavaScript fetch, OpenSSL.SSL.Connection), increasing adapter complexity
- Developers implementing new transports must understand socket-level semantics (partial writes, blocking behavior, error conditions) rather than working with higher-level stream abstractions
- Testing requires socket-level mocking infrastructure (dummyserver) rather than simpler HTTP-level request/response mocking

## Alternatives

- Use Python's high-level http.client or urllib abstractions for all HTTP operations, avoiding direct socket manipulation (rejected)
  Rejected because: High-level libraries do not provide sufficient control for HTTP/2 multiplexing, custom SSL/TLS handling (PyOpenSSL), platform-specific transports (Emscripten), or precise protocol testing (socket-level response construction in dummyserver)
  When valid: Simple HTTP/1.1 clients without custom transport requirements, SSL certificate handling, or protocol-level testing needs
- Implement separate send abstractions per transport type (socket_send, ssl_send, http2_send) without common interface (rejected)
  Rejected because: Evidence shows common interface contracts (send, recv, close, fileno) across multiple implementations, indicating intentional polymorphism; separate abstractions would prevent transport-agnostic code and complicate testing
  When valid: Codebases where transports are never substituted and no polymorphic transport handling is required
- Use asynchronous I/O (asyncio) with async send operations instead of synchronous socket send (deferred)
  Rejected because: Current evidence shows synchronous socket, ssl, and threading usage without asyncio detection; migration would require codebase-wide async/await adoption
  When valid: Future async-first HTTP client implementation or when adding async transport support alongside existing synchronous transports

## Risks

- Platform-specific send implementations (Emscripten js_xhr.send(), PyOpenSSL connection.send()) may have subtle semantic differences in error handling, partial write behavior, or encoding that violate interface contract assumptions
  Mitigation: Implement comprehensive integration tests (test_socketlevel.py, test_ssltransport.py) exercising send operations across all transport types with error injection, partial write simulation, and encoding validation
  Owner: Transport layer maintainers
- HTTP/2 multiplexing requires coordinated send operations across multiple streams; naive send implementations may cause frame interleaving corruption or deadlocks
  Mitigation: Centralize HTTP/2 send logic in http2/connection.py with explicit threading coordination (threading library detected) and stream state management; verify with test_http2_connection.py integration tests
  Owner: HTTP/2 implementation team
- Developers unfamiliar with socket-level programming may implement send operations that ignore partial writes or mishandle blocking/non-blocking socket modes
  Mitigation: Provide reference implementations (util/ssltransport.py demonstrates sendall pattern with byte_view slicing), code review checklist for send implementations, and test cases validating partial write handling
  Owner: Engineering team, code reviewers

## Implementation Notes

- Reference util/ssltransport.py for canonical send implementation pattern: handle partial writes with loop, use byte_view slicing to track progress, propagate socket errors appropriately
- When implementing new transports, ensure send() accepts bytes/bytearray/memoryview and converts strings to UTF-8 bytes explicitly (see test_socketlevel.py .encode('utf-8') pattern)
- Test send implementations with dummyserver/testcase.py infrastructure to validate protocol compliance; use socket_handler pattern for low-level response construction
- For HTTP/2 implementations, coordinate send operations with stream multiplexing using threading primitives (threading library detected in http2/connection.py); ensure frame boundaries are respected
- Platform adapters (Emscripten, PyOpenSSL) should document semantic differences between native send() and adapted APIs (js_xhr.send(), OpenSSL.SSL.Connection.send()) in module docstrings

## Continuation Context


Verify commands:
- grep -r 'def send(' src/urllib3/ --include='*.py' | grep -E '(connection|transport|ssl)' # Verify send method presence in transport implementations
- grep -r '\.send\(' test/ --include='*.py' | grep -E '(sock|conn|ssock)' # Verify test infrastructure uses socket-level send operations
- python -m pytest test/test_socketlevel.py test/test_ssltransport.py test/test_http2_connection.py -v # Validate socket-level integration tests pass

Accept when:
- All transport implementations in src/urllib3/ expose send() method accepting bytes, verified by grep showing consistent method signatures
- Integration tests (test_socketlevel.py, test_ssltransport.py, test_http2_connection.py) pass, confirming send operations work correctly across socket, SSL, and HTTP/2 transports
- Code review confirms new transport implementations handle partial writes and maintain interface contracts (send, recv, close, fileno)

## Enforcement

- Verified by: Automated integration tests in test/test_socketlevel.py, test/test_ssltransport.py, test/test_http2_connection.py validating send operations across transports
- Verified by: Code review checklist requiring verification of send() method signature, partial write handling, and encoding semantics for new transport implementations
- Verified by: CI pipeline running socket-level tests with dummyserver infrastructure to catch protocol compliance issues
- Violation handling: Integration test failures block merge until send implementation is corrected
- Violation handling: Code review identifies missing send() methods or incorrect signatures, requires revision before approval
- Violation handling: Runtime errors from incorrect send usage (partial write mishandling, encoding errors) trigger bug reports and regression test creation
- Exception process: Platform-specific adapters (Emscripten, PyOpenSSL) document semantic differences in module docstrings and obtain maintainer approval
- Exception process: Test mocks simulating send failures document deviation in test comments and limit scope to test modules
- Exception process: New transport types requiring different send semantics propose ADR amendment with evidence of technical necessity
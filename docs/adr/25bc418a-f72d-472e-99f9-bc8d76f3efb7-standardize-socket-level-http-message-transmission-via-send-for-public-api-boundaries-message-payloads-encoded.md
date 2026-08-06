# Standardize Socket-Level HTTP Message Transmission via send() for Public API Boundaries: Message Payloads Encoded

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements HTTP client functionality requiring direct socket-level control for protocol negotiation (HTTP/1.1, HTTP/2, ALPN), TLS handshakes, and custom transport layers
- Multiple transport implementations (native sockets, pyOpenSSL, Emscripten fetch, HTTP/2 connections, SSL transport wrappers) converge on socket.send() as the primitive for message transmission
- Test infrastructure requires deterministic HTTP response injection at the socket layer to validate client behavior across protocol variants, certificate handling, and streaming scenarios
- The pattern spans 8 files including production code (src/urllib3/http2/connection.py, src/urllib3/contrib/pyopenssl.py, src/urllib3/util/ssltransport.py, src/urllib3/contrib/emscripten/fetch.py) and test infrastructure (test/with_dummyserver/test_socketlevel.py, test/test_ssltransport.py, test/test_http2_connection.py, dummyserver/testcase.py)
- The abstraction enables protocol-agnostic message queuing where higher-level HTTP semantics (headers, body, chunking) are serialized to byte sequences before transmission

## Problem Statement

HTTP client libraries must support multiple transport layers (native sockets, OpenSSL, platform-specific implementations) and protocol versions (HTTP/1.1, HTTP/2) while maintaining consistent message transmission semantics. Without a standardized socket-level send interface, each transport implementation would require custom message queuing logic, leading to fragmented error handling, inconsistent buffering behavior, and duplicated serialization code across protocol adapters.

## Decision

1. MUST: Message payloads MUST be encoded to bytes (via .encode('utf-8') or equivalent) before invoking send()

## Policy Block

- MUST Message payloads MUST be encoded to bytes (via .encode('utf-8') or equivalent) before invoking send()

In scope:
- All HTTP/1.1 and HTTP/2 connection implementations in src/urllib3/
- SSL/TLS transport wrappers (SSLTransport, pyOpenSSL adapters)
- Platform-specific implementations (Emscripten fetch, native sockets)
- Test infrastructure socket handlers in test/ and dummyserver/
- Public API methods: connect, putrequest, putheader, endheaders, send

Out of scope:
- High-level request/response object construction (before serialization)
- Connection pooling and lifecycle management
- Retry logic and error recovery (above transport layer)
- HTTP parsing and deserialization (receive path)
- Application-level middleware and interceptors

Exceptions:
- EXC-001: Platform-specific implementations (e.g., Emscripten fetch) cannot access native sockets
- EXC-002: Mock/stub implementations in unit tests require simplified send() behavior

## Rationale

- Evidence shows 8 files with 92.73% confidence converging on sock.send() as the message transmission primitive across diverse transport implementations (native sockets, pyOpenSSL, HTTP/2, SSLTransport, Emscripten)
- The pattern enables protocol-agnostic abstraction where HTTP semantics are separated from transport mechanics—higher layers construct messages, send() handles delivery
- Test infrastructure demonstrates the pattern's utility: deterministic HTTP response injection via sock.send() enables validation of client behavior across protocol variants, ALPN negotiation, certificate handling, and streaming scenarios
- Standardizing on send() as the boundary operation provides a clear contract for transport layer implementations while allowing internal buffering and optimization flexibility

## Consequences

Positive:
- Consistent message transmission semantics across all transport implementations (native sockets, OpenSSL, platform-specific adapters)
- Clear separation of concerns: HTTP protocol logic constructs messages, transport layer handles delivery via send()
- Simplified test infrastructure: socket-level response injection enables comprehensive protocol testing without full server infrastructure
- Transport layer substitutability: implementations can be swapped as long as they honor send() contract

Negative:
- Low-level socket API exposure increases surface area for platform-specific bugs (blocking behavior, partial sends, error codes)
- Byte-level encoding requirements push serialization responsibility to call sites, risking encoding inconsistencies
- Test code using raw sock.send() with byte literals is brittle to HTTP protocol format changes
- Abstraction leakage: higher-level code must understand socket semantics (buffering, blocking, error handling) rather than pure message passing

## Alternatives

- Implement transport-specific write() methods with internal buffering and automatic encoding (rejected)
  Rejected because: Would fragment the codebase with transport-specific APIs, preventing uniform testing and increasing maintenance burden across HTTP/1.1, HTTP/2, OpenSSL, and platform-specific implementations
  When valid: In systems with single transport layer or where transport abstraction is not required
- Use higher-level stream abstractions (io.BytesIO, asyncio streams) as transmission boundary (rejected)
  Rejected because: Evidence shows direct socket control is required for ALPN negotiation, TLS handshake customization, and protocol-level testing; stream abstractions would obscure these low-level operations
  When valid: For application-level HTTP clients without custom TLS or protocol negotiation requirements
- Define abstract Transport interface with send() as contract method (deferred)
  Rejected because: Not rejected—this would formalize the observed pattern; deferred pending explicit interface definition
  When valid: When formalizing the implicit contract observed across implementations

## Risks

- Partial send() operations on non-blocking sockets may silently truncate messages if not handled with retry logic
  Mitigation: Implement sendall() wrapper or explicit partial-send handling in transport layers; add integration tests validating complete message delivery under backpressure
  Owner: Transport layer maintainers
- Platform-specific socket behavior (Windows vs. Unix, Emscripten) may cause inconsistent send() semantics
  Mitigation: Maintain platform-specific test suite (test/test_ssltransport.py, test/with_dummyserver/test_socketlevel.py) validating send() behavior; document platform differences
  Owner: Cross-platform compatibility team
- Direct byte encoding at call sites risks charset inconsistencies (UTF-8 vs. ASCII vs. Latin-1 for HTTP headers)
  Mitigation: Centralize encoding logic in message construction utilities; enforce encoding validation in code review and linting
  Owner: API design team

## Implementation Notes

- Wrap socket.send() calls with error handling for EAGAIN, EWOULDBLOCK, and connection errors; consider sendall() for blocking sockets
- Ensure all string data is encoded to bytes before send()—use .encode('utf-8') for body content and .encode('ascii') for HTTP headers per RFC 7230
- In test infrastructure, construct HTTP responses with explicit \r\n line endings and Content-Length headers to ensure protocol compliance
- For transport abstraction layers (SSLTransport, HTTP2Connection), delegate to underlying socket.send() rather than buffering—preserve message boundaries and ordering

## Continuation Context


Verify commands:
- grep -r '\.send(' src/urllib3/ test/ | grep -E '(sock|connection)\.send\(' | wc -l
- grep -r 'def send(' src/urllib3/ | grep -v '__pycache__'
- python -m pytest test/test_ssltransport.py test/with_dummyserver/test_socketlevel.py -v

Accept when:
- All transport implementations (HTTP/2, pyOpenSSL, SSLTransport, Emscripten) use socket.send() or connection.send() as terminal transmission operation
- Test infrastructure socket handlers construct responses via sock.send() with complete HTTP/1.1 formatted messages
- No direct socket write operations bypass the send() primitive in public API methods (connect, putrequest, putheader, endheaders, send)

## Enforcement

- Verified by: Automated grep patterns in CI checking for .send() usage in transport implementations
- Verified by: Integration test suite (test/test_ssltransport.py, test/with_dummyserver/test_socketlevel.py) validating socket-level message transmission
- Verified by: Code review checklist requiring send() primitive for new transport implementations
- Violation handling: CI build failure if transport implementations bypass send() primitive
- Violation handling: Code review rejection for direct socket write operations outside send() contract
- Violation handling: Regression test addition required for any transport layer changes
- Exception process: Submit architecture review request documenting platform-specific constraints preventing send() usage
- Exception process: Provide equivalent message queuing semantics and boundary preservation guarantees
- Exception process: Obtain approval from transport layer maintainers and update exception registry
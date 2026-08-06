# Standardize Logging at Message Queue Boundaries: Implementations Use Additional

These rules are ALWAYS ACTIVE for all modules implementing message queue operations, protocol transmission boundaries, and send() methods in HTTP/2 connection handling, SSL/TLS socket wrapping, and protocol adapter implementations.

### Rules

- **R-MQ-001** MAY: Implementations MAY use additional logging for pre-transmission validation or post-transmission confirmation.
- **R-MQ-002** MUST: Initialize module-level logger immediately after imports using `logger = logging.getLogger(__name__)` in all modules implementing send() methods or message queue boundaries.
- **R-MQ-003** SHOULD: Log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions in send() methods.
- **R-MQ-004** SHOULD: Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled.
- **R-MQ-005** SHOULD: Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems.
- **R-MQ-006** MUST: Implement log sanitization utilities and code review guidelines to prevent logging of sensitive payloads (authentication tokens, PII, credentials).
- **R-MQ-007** SHOULD: Use DEBUG level for routine operations and implement sampling or rate-limiting for high-frequency events to avoid excessive log volume.

### Verify

```bash
# Verify logging infrastructure co-occurs with transmission methods
grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l

# Verify send() methods have corresponding logger initialization
grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'

# Verify logging output at transmission boundaries with DEBUG level
python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20
```

**Accept when:**
- All modules implementing send() methods or message queue boundaries contain logging.getLogger(__name__) initialization
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules
- No sensitive data (credentials, tokens, PII) appears in logged transmission payloads
- Logging statements use lazy formatting and appropriate log levels (DEBUG for routine, ERROR for exceptions)

<enforcement>
Claude Code MUST NOT skip or defer verification. All modules with send() methods or transmission boundaries MUST be checked for logger initialization and appropriate logging patterns before acceptance.
</enforcement>
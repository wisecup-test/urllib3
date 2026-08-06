# Standardize Logging at Message Queue Boundaries: Modules That Implement

These rules are ALWAYS ACTIVE for modules that implement message queue or data transmission boundaries, including HTTP/2 connection handling, SSL/TLS socket wrapping, protocol adapters, and any module defining send() methods for data transmission.

### Rules

- **R-MQB-001** MUST: Modules that implement message queue or data transmission boundaries MUST initialize a module-level logger using `logging.getLogger(__name__)` immediately after imports.
- **R-MQB-002** MUST: For send() methods, log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions.
- **R-MQB-003** SHOULD: Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled.
- **R-MQB-004** SHOULD: Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems.
- **R-MQB-005** MUST: Implement log sanitization utilities and code review guidelines to prevent logging of sensitive payloads (authentication tokens, PII, credentials).

### Verify

```bash
# Verify logger initialization in transmission boundary modules
grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l

# Verify co-occurrence of logging with send() methods
grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'

# Test execution with DEBUG logging enabled
python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20
```

**Accept when:**
- All modules implementing send() methods or message queue boundaries contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules
- No sensitive data (credentials, tokens, PII) is logged in transmission payloads

<enforcement>
Claude Code MUST NOT skip or defer verification. All modules implementing transmission boundaries MUST be audited for logger initialization and sensitive data logging before acceptance.
</enforcement>
# Standardize Logging at Message Queue Boundaries: Logging Statements Transmission

These rules are ALWAYS ACTIVE for modules implementing message queue operations, HTTP/2 connection handling, SSL/TLS socket wrapping, and any module that defines send() methods for data transmission at protocol boundaries.

### Rules

- **R-LOG-001** SHOULD: Logging statements at transmission boundaries SHOULD include contextual information such as data size, connection state, or protocol-specific metadata.
- **R-LOG-002** MUST: Initialize the module-level logger immediately after imports using `logger = logging.getLogger(__name__)` to ensure it is available throughout the module.
- **R-LOG-003** SHOULD: For send() methods, log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions.
- **R-LOG-004** SHOULD: Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled.
- **R-LOG-005** MAY: Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems.

### Verify

```bash
# Verify logging.getLogger(__name__) initialization in transmission boundary modules
grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l

# Verify co-occurrence of logging infrastructure with send() methods
grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'

# Verify observable log output from transmission boundary modules
python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20
```

**Accept when:**
- All modules implementing send() methods or message queue boundaries contain logging.getLogger(__name__) initialization
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules
- No sensitive data (credentials, tokens, PII) is logged in transmission payloads

<enforcement>
Clause Code MUST NOT skip or defer verification. All modules at transmission boundaries MUST initialize module-level loggers and include contextual information in logging statements before code review approval.
</enforcement>
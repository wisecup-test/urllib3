# Standardize Logging at Message Queue Boundaries: Modules Implementing Protocol

These rules are ALWAYS ACTIVE for all modules implementing protocol-level send operations, HTTP/2 connection handling, SSL/TLS socket wrapping, and any module that defines send() methods for data transmission at message queue or I/O boundaries.

### Rules

- **R-MQ-001** SHOULD: Modules implementing protocol-level send operations SHOULD log at appropriate levels (DEBUG for routine operations, WARNING for recoverable issues, ERROR for failures).
- **R-MQ-002** SHOULD: Initialize module-level logger immediately after imports using `logger = logging.getLogger(__name__)` to ensure availability throughout the module.
- **R-MQ-003** SHOULD: Log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions in send() methods.
- **R-MQ-004** SHOULD: Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled.
- **R-MQ-005** SHOULD: Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems.
- **R-MQ-006** MUST NOT: Log sensitive data (authentication tokens, PII, credentials) at transmission boundaries without sanitization.

### Verify

```bash
# Check for logging infrastructure in modules with send() methods
grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l

# Verify co-occurrence of logging with send() method definitions
grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'

# Test execution with DEBUG logging to verify observable output
python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20
```

**Accept when:**
- All modules implementing send() methods or message queue boundaries contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules
- No sensitive data (tokens, credentials, PII) is logged in transmission payloads

<enforcement>
Claude Code MUST NOT skip or defer verification. All modules implementing protocol-level send operations MUST be reviewed for compliance with logging standards before merge. Static analysis and CI pipeline checks are mandatory.
</enforcement>
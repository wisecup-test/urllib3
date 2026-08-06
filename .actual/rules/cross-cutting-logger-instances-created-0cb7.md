# Standardize Logging at Message Queue Boundaries: Logger Instances Created

These rules are ALWAYS ACTIVE for modules implementing message queue operations, HTTP/2 connection handling, SSL/TLS socket wrapping, protocol adapters, and any module that defines send() methods for data transmission.

### Rules

- **R-LOG-001** MUST: Logger instances MUST be created at module scope before any send() or transmission methods are defined.
- **R-LOG-002** MUST: Initialize the module-level logger immediately after imports using `logger = logging.getLogger(__name__)` to ensure it's available throughout the module.
- **R-LOG-003** SHOULD: For send() methods, log at DEBUG level before transmission with context (e.g., data size, destination) and at ERROR level for exceptions.
- **R-LOG-004** SHOULD: Use lazy string formatting (logger.debug('Message: %s', value)) rather than f-strings to avoid string construction overhead when logging is disabled.
- **R-LOG-005** MAY: Consider using structured logging (JSON format) for machine-readable logs that can be parsed by log aggregation systems.

### Verify

```bash
# Verify logger initialization in transmission boundary modules
grep -r 'logging\.getLogger(__name__)' --include='*.py' | grep -E '(send|connection|queue)' | wc -l

# Verify co-occurrence of logging infrastructure with send() methods
grep -r 'def send\(' --include='*.py' -A 20 | grep -c 'logging\.getLogger'

# Verify observable log output from transmission boundary modules
python -m pytest tests/ -v --log-level=DEBUG 2>&1 | grep -E '(send|transmission|queue)' | head -20
```

**Accept when:**
- All modules implementing send() methods or message queue boundaries contain logging.getLogger(__name__) initialization at module scope.
- Grep verification shows consistent co-occurrence of logging infrastructure with transmission methods.
- Test execution with DEBUG logging enabled produces observable log output from transmission boundary modules.
- No send() methods exist in transmission boundary modules without corresponding logger initialization.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for modules in scope. Code review and CI pipeline checks MUST enforce logger initialization before merge.
</enforcement>
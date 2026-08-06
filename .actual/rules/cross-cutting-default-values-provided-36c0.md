# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Default Values Provided

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-DICT-001** MUST: Default values provided to `.get()` MUST match the semantic meaning of absence for that specific context (e.g., False for boolean flags, None for optional strings, 80 for default HTTP port).
- **R-DICT-002** MUST: All HTTP header access in production code MUST use `.get()` method rather than direct key access for external data boundaries.
- **R-DICT-003** MUST: Environment variable access via `os.environ` MUST consistently use `.get()` with appropriate defaults.
- **R-DICT-004** SHOULD: Establish canonical default values for common protocol-level keys: `port_by_scheme.get(scheme, 80)` for HTTP, `port_by_scheme.get(scheme, 443)` for HTTPS.
- **R-DICT-005** SHOULD: For boolean configuration flags, use False as default (e.g., `self.headers.get(sort_key, False)`) to enable opt-in behavior.
- **R-DICT-006** SHOULD: For optional string headers like Retry-After or Location, use `.get()` without default argument to return None, then explicitly check for None before processing.
- **R-DICT-007** SHOULD: Document in API contracts whether None is a valid return value or whether calling code must handle it as an error condition.
- **R-DICT-008** MAY: Use type hints with `Optional[T]` for `.get()` return values to clarify None-handling expectations.

### Verify

```bash
# Count .get() usage on known external data structures
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct dictionary access patterns that should use .get()
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite to verify no KeyError exceptions from header/config access
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -i 'keyerror.*header\|keyerror.*environ\|keyerror.*cert' | wc -l
```

**Accept when:**
- All HTTP header access in production code uses `.get()` method rather than direct key access
- Environment variable access via `os.environ` consistently uses `.get()` with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Default values for common keys are documented and consistently applied across the codebase
- Type hints with `Optional[T]` are present for `.get()` return values in critical paths

<enforcement>
Clause Code MUST NOT skip or defer verification. All dictionary access at external boundaries (HTTP headers, environment variables, certificate fields, protocol mappings) MUST use `.get()` with semantically appropriate default values. Violations require code review feedback and explicit exception documentation before merge approval.
</enforcement>
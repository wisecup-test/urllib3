# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Protocol Mapping Dictionaries

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-DICT-001** MUST: Protocol mapping dictionaries (port_by_scheme, HASHFUNC_MAP, _SSL_VERSION_TO_TLS_VERSION) MUST be accessed via .get() with sensible fallback defaults.
- **R-DICT-002** MUST: HTTP response header access (response.headers.get, returned_headers.get, self.headers.get) MUST use .get() method rather than direct key access to prevent KeyError exceptions.
- **R-DICT-003** MUST: Environment variable access (os.environ.get) MUST use .get() method with appropriate defaults for CI detection and SSL configuration.
- **R-DICT-004** MUST: Certificate field access during SSL hostname matching (cert.get) MUST use .get() method to handle missing optional fields.
- **R-DICT-005** SHOULD: Establish canonical default values for common protocol-level keys: port_by_scheme.get(scheme, 80) for HTTP, port_by_scheme.get(scheme, 443) for HTTPS.
- **R-DICT-006** SHOULD: For boolean configuration flags, use False as default (e.g., self.headers.get(sort_key, False)) to enable opt-in behavior.
- **R-DICT-007** SHOULD: For optional string headers like Retry-After or Location, use .get() without default argument to return None, then explicitly check for None before processing.
- **R-DICT-008** SHOULD: Document in API contracts whether None is a valid return value or whether calling code must handle it as an error condition.
- **R-DICT-009** MAY: Direct dictionary key access (dict[key]) or 'in' operator checks are permitted only when EXC-001 (guaranteed key presence by construction) or EXC-002 (intentional KeyError handling) conditions are met and documented.

### Verify

```bash
# Count .get() usage on headers, environ, cert, and kwargs
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct dictionary access patterns on external data structures
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite and verify no KeyError from header/config access
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -c 'KeyError.*headers\|KeyError.*environ\|KeyError.*cert' || echo "0"
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Protocol mapping dictionaries are accessed with sensible fallback defaults
- Type hints with Optional[T] are present for .get() return values in critical paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All dictionary access on external data structures (HTTP headers, environment variables, certificate fields, protocol mappings) MUST use .get() method with appropriate defaults. Exceptions require inline documentation explaining guaranteed key presence or intentional KeyError handling, plus code review approval.
</enforcement>
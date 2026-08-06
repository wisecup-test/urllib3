# Standardize Dictionary .get() Method for Safe Header and Configuration Access: Use Get Without

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP response header access in connection pooling and response handling, environment variable access for CI detection and SSL key logging configuration, protocol mapping lookups for ports, hash functions, and SSL/TLS versions, certificate field access during SSL hostname matching, configuration parameter access in retry logic and proxy handling, and test infrastructure header validation and assertion logic.

### Rules

- **R-DICT-001** MAY: APIs MAY use .get() without a default argument when None is the appropriate semantic value for missing keys.
- **R-DICT-002** MUST: Use .get() method for all HTTP header access at external boundaries (response.headers.get, returned_headers.get, self.headers.get) rather than direct key access.
- **R-DICT-003** MUST: Use os.environ.get() with appropriate defaults for all environment variable access rather than direct dictionary access.
- **R-DICT-004** MUST: Use .get() for protocol mapping lookups (port_by_scheme.get, HASHFUNC_MAP.get, cert.get) to handle missing keys defensively.
- **R-DICT-005** SHOULD: Establish canonical default values for common protocol-level keys: port_by_scheme.get(scheme, 80) for HTTP, port_by_scheme.get(scheme, 443) for HTTPS.
- **R-DICT-006** SHOULD: Use False as default for boolean configuration flags (e.g., self.headers.get(sort_key, False)) to enable opt-in behavior.
- **R-DICT-007** SHOULD: For optional string headers like Retry-After or Location, use .get() without default argument to return None, then explicitly check for None before processing.
- **R-DICT-008** MUST NOT: Use direct dictionary key access (dict[key]) for external data structures (HTTP headers, environment variables, client responses) without exception handling.
- **R-DICT-009** MUST NOT: Raise KeyError exceptions at runtime when accessing optional HTTP headers, environment variables, and configuration parameters.
- **R-DICT-EXC-001** Exception: Direct dictionary access is permitted when the dictionary is constructed internally with guaranteed key presence and the code path ensures the key exists.
- **R-DICT-EXC-002** Exception: Direct dictionary access is permitted when KeyError exception is explicitly caught and handled as part of the error handling strategy.

### Verify

```bash
# Count .get() usage on headers, environ, cert, kwargs across production and test code
grep -r '\.get(' src/urllib3/ test/ dummyserver/ | grep -E '(headers|environ|cert|kwargs)' | wc -l

# Count direct dictionary access patterns on external data structures
grep -r '\["' src/urllib3/ | grep -E '(headers|environ|cert)\[' | wc -l

# Run test suite and verify no KeyError exceptions from header/config access
python -m pytest test/ -v -k 'test_' --tb=short 2>&1 | grep -c 'KeyError.*headers\|KeyError.*environ\|KeyError.*cert' || echo "0"
```

**Accept when:**
- All HTTP header access in production code uses .get() method rather than direct key access
- Environment variable access via os.environ consistently uses .get() with appropriate defaults
- No KeyError exceptions are raised from dictionary access at external boundaries (headers, environment, client responses) during test suite execution
- Protocol mapping lookups (port_by_scheme, HASHFUNC_MAP, cert fields) use .get() defensively
- Test infrastructure header validation uses .get() to allow omission of optional headers without triggering exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST execute successfully before accepting code changes. Code review MUST check for .get() usage on external data structures and flag direct key access for remediation. Static analysis via grep or AST-based linting MUST detect and warn about direct dictionary access patterns on known external data structures. Violations require code review feedback requesting change to .get() method before merge approval.
</enforcement>
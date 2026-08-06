# Use .get() Method with Default Values for Safe External Header and Environment Access: Internal Data Structures

These rules are ALWAYS ACTIVE for all code accessing external HTTP headers, environment variables, and certificate dictionaries that may contain optional or missing keys.

### Rules

- **R-EXT-001** MUST: Use `.get()` method with appropriate default values when accessing HTTP response headers from external servers.
- **R-EXT-002** MUST: Use `.get()` method with appropriate default values when accessing HTTP request headers received from external clients.
- **R-EXT-003** MUST: Use `.get()` method with appropriate default values when reading environment variables from `os.environ`.
- **R-EXT-004** MUST: Use `.get()` method with appropriate default values when accessing SSL certificate dictionaries from external TLS handshakes.
- **R-EXT-005** MUST: Use `.get()` method with appropriate default values when accessing configuration lookup tables with optional keys.
- **R-EXT-006** MAY: Internal data structures with guaranteed key presence MAY use direct dictionary access without `.get()`.
- **R-EXT-007** MUST: Provide sensible default values that match the expected type: `None` for optional headers, empty string or `False` for environment variables, empty tuple `()` for certificate fields, and type-consistent fallbacks for lookup tables.
- **R-EXT-008** MUST NOT: Use direct dictionary access (e.g., `headers['key']` or `os.environ['KEY']`) without `.get()` unless the key presence is explicitly validated and KeyError is handled in an exception handler (EXC-001).

### Verify

```bash
# Check for direct header access without .get()
grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l

# Check for direct environ access without .get()
grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l

# Verify .get() usage for external data structures
grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l
```

**Accept when:**
- All HTTP header access in `src/urllib3/` uses `.get()` method with appropriate defaults
- All `os.environ` access uses `.get()` method with appropriate defaults
- All certificate field access uses `.get()` method with appropriate defaults
- No KeyError exceptions occur in production logs related to header or environment variable access
- Default values are type-consistent with their corresponding present values
- Optional headers default to `None`, environment variables default to empty string or `False`, certificate fields default to empty tuple `()`

<enforcement>
Clause Code MUST NOT skip or defer verification. All new code accessing external headers, environment variables, or certificate dictionaries MUST use `.get()` with documented default values. CI pipeline MUST fail if direct dictionary access patterns are detected. Code review MUST block merges violating this rule unless explicit exception (EXC-001) is documented and approved.
</enforcement>
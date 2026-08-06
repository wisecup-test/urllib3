# Use .get() Method with Default Values for Safe External Header and Environment Access: Access Environment Variables

These rules are ALWAYS ACTIVE for all code that accesses external HTTP headers, request headers, environment variables, SSL certificate dictionaries, and configuration lookup tables where keys may be absent.

### Rules

- **R-ENV-001** MUST: All access to environment variables from external runtime contexts MUST use `os.environ.get()` with a default value.
- **R-ENV-002** MUST: All access to HTTP response headers received from external servers MUST use `.get()` method with appropriate default values.
- **R-ENV-003** MUST: All access to HTTP request headers received from external clients MUST use `.get()` method with appropriate default values.
- **R-ENV-004** MUST: All access to SSL certificate dictionaries from external TLS handshakes MUST use `.get()` method with appropriate default values.
- **R-ENV-005** MUST: All access to configuration lookup tables with optional keys MUST use `.get()` method with appropriate default values.
- **R-ENV-006** SHOULD: For HTTP headers, use `None` as default when absence is semantically meaningful (e.g., `response.headers.get('Retry-After')` returns `None` to indicate no retry guidance).
- **R-ENV-007** SHOULD: For environment variables, use empty string or `False` as default based on downstream boolean vs. string usage (e.g., `os.environ.get('CI')` for boolean checks).
- **R-ENV-008** SHOULD: For certificate fields, use empty tuple `()` as default to enable safe iteration (e.g., `cert.get('subjectAltName', ())`).
- **R-ENV-009** SHOULD: For lookup tables and mappings, provide fallback values that maintain type consistency (e.g., `port_by_scheme.get(scheme, 80)` for integer ports).
- **R-ENV-010** MAY: Direct dictionary access without `.get()` is permitted only when code explicitly validates key presence before access and handles `KeyError` in an exception handler (EXC-001).

### Verify

```bash
# Check for direct header access without .get()
grep -r '\.headers\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l
# Expected: 0 or minimal

# Check for direct os.environ access without .get()
grep -r 'os\.environ\[' src/ test/ --include='*.py' | grep -v '.get(' | wc -l
# Expected: 0 or minimal

# Verify .get() usage for external data access
grep -r '\.get(' src/urllib3/ --include='*.py' | grep -E '(headers|environ|cert)' | wc -l
# Expected: matches pattern usage across 14+ files
```

**Accept when:**
- All HTTP header access in `src/urllib3/` uses `.get()` method with appropriate defaults
- All `os.environ` access uses `.get()` method with appropriate defaults
- No `KeyError` exceptions occur in production logs related to header or environment variable access
- Default values are type-consistent with actual values when keys are present
- Optional vs. required fields are clearly documented

<enforcement>
Clause Code MUST NOT skip or defer verification. Static analysis in CI pipeline MUST check for direct dictionary access patterns (`[key]`) on headers, environ, and cert objects. Code review MUST block merges that violate R-ENV-001 through R-ENV-009 unless EXC-001 is explicitly documented and approved. Runtime monitoring MUST alert on `KeyError` exceptions from header or environment access.
</enforcement>
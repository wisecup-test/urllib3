# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Modules Configure Additional

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring observability during integration testing.

### Rules

- **R-LOG-001** MUST: Initialize module-level loggers using `logging.getLogger(__name__)` at the top of each Python module after imports.
- **R-LOG-002** MUST: Place logger initialization before the first function or class definition in existing modules without logging.
- **R-LOG-003** MAY: Modules MAY configure additional logging handlers or formatters specific to their testing requirements.
- **R-LOG-004** SHOULD: Use `logger.debug()` for detailed state transitions, `logger.info()` for integration boundary crossings, `logger.warning()` for retry attempts, and `logger.error()` for failures.
- **R-LOG-005** SHOULD: Document exception rationale in module docstring if module-level initialization is not possible due to circular imports or other constraints.

### Verify

```bash
# Count modules with logging.getLogger(__name__) pattern
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Count modules with logging import
grep -r 'import logging' src/ | wc -l

# Verify integration tests capture hierarchical logger names
python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20

# Check for hardcoded logger names or shared logger instances
grep -r 'getLogger(["\x27][^_]' src/ | grep -v '__name__'
```

**Accept when:**
- All Python modules in src/ directory that require observability contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names
- No hardcoded logger names or shared logger instances are found in production code

<enforcement>
Clause Code MUST NOT skip or defer verification. All new modules MUST include proper logger initialization before merge. CI pipeline MUST fail if modules lack logging.getLogger(__name__) pattern. Code review MUST request logger initialization additions for non-compliant modules.
</enforcement>
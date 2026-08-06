# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Logger Instances Stored

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring integration testing observability.

### Rules

- **R-LOG-001** MUST: Logger instances MUST be initialized using `logging.getLogger(__name__)` at module level.
- **R-LOG-002** SHOULD: Logger instances SHOULD be stored in a module-level variable named 'log' or 'logger' for consistency.
- **R-LOG-003** MUST: The logging module MUST be imported at the top of each Python module that initializes a logger.
- **R-LOG-004** SHOULD: Logger initialization SHOULD occur before the first function or class definition in a module.
- **R-LOG-005** SHOULD: Logger names SHOULD reflect the module structure via `__name__` to enable hierarchical log filtering.

### Verify

```bash
# Count modules with logging.getLogger(__name__) pattern
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Count modules with logging import
grep -r 'import logging' src/ | wc -l

# Verify logger names match module structure in integration tests
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
Claude Code MUST NOT skip or defer verification. All new Python modules in scope MUST include module-level logger initialization before merge. CI pipeline MUST fail if new modules lack proper logger initialization.
</enforcement>
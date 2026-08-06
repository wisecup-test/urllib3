# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Modules Not Use

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring observability during integration testing.

### Rules

- **R-LOG-001** MUST NOT: Modules MUST NOT use hardcoded logger names or shared logger instances across unrelated modules.
- **R-LOG-002** MUST: All Python modules in src/ directory requiring observability MUST initialize module-level loggers using `logging.getLogger(__name__)`.
- **R-LOG-003** MUST: Logger initialization MUST occur at module level after imports, before first function or class definition.
- **R-LOG-004** SHOULD: Use logger.debug() for detailed state transitions, logger.info() for integration boundary crossings, logger.warning() for retry attempts, and logger.error() for failures.

### Verify

```bash
# Count existing logging.getLogger(__name__) usage
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Count logging imports
grep -r 'import logging' src/ | wc -l

# Verify specific modules have proper logger initialization
grep -E '(connectionpool\.py|util/retry\.py|contrib/emscripten/response\.py)' <(grep -r 'logging\.getLogger(__name__)' src/)

# Run integration tests with debug logging to verify hierarchical logger names
python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20
```

**Accept when:**
- All Python modules in src/ directory that require observability contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names
- No hardcoded logger names or shared logger instances are found across unrelated modules
- Logger names match module structure (e.g., urllib3.connectionpool, urllib3.util.retry)

<enforcement>
Claude Code MUST NOT skip or defer verification. All new Python modules in scope MUST include proper logger initialization before merge. CI pipeline MUST fail if modules lack logging.getLogger(__name__) pattern. Code review MUST request logger initialization additions for non-compliant modules.
</enforcement>
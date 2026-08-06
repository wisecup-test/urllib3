# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Modules Import Both

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring integration testing observability.

### Rules

- **R-LOG-001** MUST: Modules MUST import both 'logging' and use 'getLogger' to obtain logger instances.
- **R-LOG-002** MUST: Module-level logger initialization MUST use `logging.getLogger(__name__)` pattern at the top of each module after imports.
- **R-LOG-003** MUST: Logger initialization MUST occur before the first function or class definition in existing modules without logging.
- **R-LOG-004** SHOULD: Use logger.debug() for detailed state transitions, logger.info() for integration boundary crossings, logger.warning() for retry attempts, and logger.error() for failures.
- **R-LOG-005** MAY: Modules may document exception rationale in module docstring if module-level initialization is not possible due to circular imports or other constraints.

### Verify

```bash
# Count modules with logging.getLogger(__name__) pattern
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Count modules with logging import
grep -r 'import logging' src/ | wc -l

# Verify pattern in key modules
grep -r 'logging\.getLogger(__name__)' src/ | grep -E '(connectionpool|util/retry|contrib/emscripten/response)'

# Run integration tests with debug logging
python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20
```

**Accept when:**
- All Python modules in src/ directory that require observability contain 'logging.getLogger(__name__)' initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names
- New modules added to the codebase include proper logger initialization before merge

<enforcement>
Claude Code MUST NOT skip or defer verification. All new Python modules in scope MUST include logging.getLogger(__name__) initialization. CI pipeline MUST fail if new modules lack proper logger initialization. Code review MUST request addition of logging.getLogger(__name__) before merge if missing.
</enforcement>
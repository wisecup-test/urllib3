# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Each Python Module

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring observability during integration testing.

### Rules

- **R-LOG-001** MUST: Each Python module MUST initialize its logger using `logging.getLogger(__name__)` at module level, placed after all imports and before the first function or class definition.

### Verify

```bash
# Count modules with proper logger initialization
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Verify logging module is imported in files with logger initialization
grep -r 'import logging' src/ | wc -l

# Run integration tests with debug logging to verify hierarchical logger names
python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20
```

**Accept when:**
- All Python modules in src/ directory that require observability contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names
- Logger initialization appears at module level after imports and before first function or class definition

<enforcement>
Claude Code MUST NOT skip or defer verification. All new Python modules in scope MUST include module-level logger initialization before merge. CI pipeline MUST fail if new modules lack proper logger initialization.
</enforcement>
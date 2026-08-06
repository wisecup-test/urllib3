# Standardize logging.getLogger(__name__) for Module-Level Logger Initialization: Logging Statements Placed

These rules are ALWAYS ACTIVE for all Python modules in the src/ directory that require observability, including connection pooling components, retry logic, HTTP response processing, and any module requiring observability during integration testing.

### Rules

- **R-LOG-001** SHOULD: Logging statements SHOULD be placed at integration boundaries where external clients are accessed or internal state transitions occur.
- **R-LOG-002** MUST: Each Python module requiring observability MUST initialize a module-level logger using `logging.getLogger(__name__)` after imports and before function or class definitions.
- **R-LOG-003** SHOULD: Use `logger.debug()` for detailed state transitions, `logger.info()` for integration boundary crossings, `logger.warning()` for retry attempts, and `logger.error()` for failures.
- **R-LOG-004** MUST: All new Python modules in src/ MUST include `import logging` and `logger = logging.getLogger(__name__)` at the module level.

### Verify

```bash
# Count existing logger initializations
grep -r 'logging\.getLogger(__name__)' src/ | wc -l

# Count logging imports
grep -r 'import logging' src/ | wc -l

# Verify integration test log capture
python -m pytest tests/ -v --log-cli-level=DEBUG 2>&1 | grep -E '(urllib3\.(connectionpool|util\.retry|contrib\.emscripten\.response))' | head -20

# Check for hardcoded logger names or shared instances
grep -r 'getLogger(' src/ | grep -v '__name__' | grep -v '#'
```

**Accept when:**
- All Python modules in src/ directory that require observability contain `logging.getLogger(__name__)` initialization
- Grep verification shows consistent usage across connectionpool.py, util/retry.py, and contrib/emscripten/response.py
- Integration tests can capture and assert on log output from specific modules using hierarchical logger names
- No hardcoded logger names or shared logger instances are found in production code

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST require logger initialization in new modules. CI pipeline MUST fail if new modules lack proper logger initialization. Linting rules MUST flag hardcoded logger names or shared logger instances.
</enforcement>
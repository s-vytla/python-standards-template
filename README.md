# Python Standards Template

A **standards injection layer** for Python projects using [Copier](https://copier.readthedocs.io/). This template manages only tooling infrastructure and can be safely applied to both new and mature repositories without disrupting existing code.

## What This Template Does

This template enforces organizational Python standards while preserving developer autonomy:

**Manages:**
- Linting configuration (Ruff)
- Type checking configuration (mypy)
- Testing configuration (pytest)
- Pre-commit hooks
- CI/CD workflows (GitHub Actions)
- Development commands (run.sh)
- Optional Docker configuration

**Never Touches:**
- Your source code
- Package metadata (`pyproject.toml` [project] section)
- Documentation
- Business logic
- Project structure

## Quick Start

### For New Projects

```bash
# Create project directory
mkdir my-project && cd my-project

# Apply template
copier copy gh:s-vytla/python-standards-template . --trust

# Initialize git (if not already done)
git init
git add .
git commit -m "Initial commit with Python standards"

# Install development dependencies
./run.sh install_dev
```

### For Existing Projects

```bash
# Navigate to your project
cd my-existing-project

# Apply template (will not overwrite pyproject.toml or README.md)
copier copy gh:s-vytla/python-standards-template . --trust

# Review changes
git status
git diff

# Install pre-commit hooks
./run.sh install_dev

# Run quality checks
./run.sh lint
./run.sh type_check
./run.sh test
```

## Template Configuration

When you apply the template, you'll be asked:

- **Python version**: 3.11, 3.12, or 3.13 (default: 3.12)
- **Line length**: Maximum line length for formatting (default: 88)
- **Strict mypy**: Enable strict type checking (default: true for new projects)
- **Use Docker**: Include Dockerfile (default: false)
- **Use GitHub Actions**: Include CI workflow (default: true)

## Files Created

**Always created:**
- `ruff.toml` - Linting and formatting configuration
- `mypy.ini` - Type checking configuration
- `pytest.ini` - Testing configuration
- `.pre-commit-config.yaml` - Pre-commit hooks
- `run.sh` - Standard development commands

**Conditionally created:**
- `Dockerfile` - Only if you enable Docker support
- `.github/workflows/ci.yml` - Only if you enable GitHub Actions

**Never overwritten:**
- `pyproject.toml` - Protected by `_skip_if_exists`
- `README.md` - Protected by `_skip_if_exists`
- Any source code files

## Available Commands

The template provides a standard run.sh interface:

```bash
./run.sh help          # Show all available commands
./run.sh install       # Install production dependencies
./run.sh install_dev   # Install development dependencies and pre-commit hooks
./run.sh lint          # Run linting checks
./run.sh format        # Format code with auto-fix
./run.sh type_check    # Run type checking
./run.sh test          # Run tests without coverage
./run.sh test_cov      # Run tests with coverage report
./run.sh clean         # Remove all generated files and caches
```

## Updating Standards

As organizational standards evolve, update your project:

```bash
# Update to latest template version
copier update

# Review changes
git diff

# Commit updates
git add .
git commit -m "Update Python standards"
```

Copier preserves your customizations and only updates template-managed files.

## Customization

### Ruff Configuration

Edit `ruff.toml` to customize linting rules:

```toml
[lint]
# Add project-specific ignores
ignore = [
    "E501",  # Line too long (if you need longer lines)
]

# Add project-specific per-file ignores
[lint.per-file-ignores]
"scripts/*" = ["T201"]  # Allow print in scripts
```

### Mypy Configuration

Edit `mypy.ini` to adjust type checking:

```ini
# Add type stubs for your dependencies
[mypy-your_library.*]
ignore_missing_imports = True
```

### Pre-commit Hooks

Edit `.pre-commit-config.yaml` to add hooks or dependencies:

```yaml
- repo: https://github.com/pre-commit/mirrors-mypy
  rev: v1.11.0
  hooks:
    - id: mypy
      additional_dependencies:
        - types-requests
        - types-PyYAML
```

### CI/CD Workflow

Edit `.github/workflows/ci.yml` to add steps:

```yaml
- name: Run integration tests
  run: pytest tests/integration
```

## Strict vs Relaxed Type Checking

### Strict Mode (Recommended for New Projects)

Enforces comprehensive type annotations:
- All functions must have type hints
- No implicit `Any` types
- Strict equality checking
- Full type safety guarantees

### Relaxed Mode (For Existing Projects)

Enables gradual typing adoption:
- Type hints optional
- Allows untyped code
- Fewer breaking changes
- Incremental improvement path

Switch modes by updating the template:

```bash
copier update --defaults
# Change strict_mypy answer when prompted
```

## Testing Strategy

The template configures pytest with useful markers:

```python
import pytest

@pytest.mark.unit
def test_fast_unit():
    """Fast, isolated unit test"""
    pass

@pytest.mark.integration
def test_with_database():
    """Integration test with external resources"""
    pass

@pytest.mark.slow
def test_performance():
    """Long-running test"""
    pass
```

Run specific test categories:

```bash
pytest -m unit              # Only unit tests
pytest -m "not slow"        # Exclude slow tests
pytest -m "unit or smoke"   # Multiple markers
```

## CI/CD Pipeline

If you enabled GitHub Actions, the template provides:

**Three separate jobs:**
1. **Lint**: Ruff linting and formatting checks
2. **Type Check**: Mypy type checking
3. **Test**: Pytest across Python 3.11, 3.12, 3.13

**Triggers:**
- Push to main/master/develop branches
- All pull requests

**Features:**
- Matrix testing across Python versions
- Coverage upload to Codecov (optional)
- Fast feedback with parallel jobs

## Docker Support

If you enabled Docker, the template provides a production-ready Dockerfile:

**Features:**
- Multi-stage build for smaller images
- Non-root user for security
- Virtual environment isolation
- Minimal base image (python:slim)

**Usage:**

```bash
# Build image
docker build -t my-app .

# Run container
docker run my-app

# Override command
docker run my-app python -m your_module.cli
```

**Customize:**

Edit `Dockerfile` to:
- Change the CMD to your entry point
- Add runtime dependencies
- Adjust health check
- Configure environment variables

## Troubleshooting

### Pre-commit hooks fail on first commit

**Symptom**: Hooks modify files, commit is rejected

**Solution**: This is expected. Run `git add .` and commit again.

```bash
git commit -m "Initial commit"  # May fail with auto-fixes
git add .                       # Stage the fixes
git commit -m "Initial commit"  # Should succeed
```

### Mypy reports many errors in existing project

**Symptom**: Hundreds of type errors after applying template

**Solution**: Use relaxed mode for existing projects:

```bash
copier update
# Answer "false" to strict_mypy question
```

Or add gradual `# type: ignore` comments:

```python
result = legacy_function()  # type: ignore[no-untyped-call]
```

### Ruff conflicts with existing style

**Symptom**: Ruff formatting changes your preferred style

**Solution**: Customize `ruff.toml`:

```toml
[format]
quote-style = "single"  # If you prefer single quotes
skip-magic-trailing-comma = true  # If you don't want trailing commas
```

### Tests not discovered

**Symptom**: `pytest` finds no tests

**Solution**: Ensure test structure matches configuration:

```
your-project/
├── tests/           # Test directory name
│   ├── __init__.py
│   └── test_*.py    # Test file pattern
```

Or customize `pytest.ini`:

```ini
[pytest]
testpaths = my_tests  # If you use different directory name
```

### CI fails but local tests pass

**Symptom**: GitHub Actions fails but `./run.sh test` succeeds

**Solution**: Ensure all dependencies in pyproject.toml:

```toml
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "mypy>=1.0",
    "ruff>=0.6",
    # Add all test dependencies
]
```

## Requirements

- Python 3.11+
- [Copier](https://copier.readthedocs.io/) 9.0+
- Git

Install Copier:

```bash
pip install copier
# or
pipx install copier
```

## Philosophy

This template follows key principles:

1. **Infrastructure Only**: Manages tooling, never business logic
2. **Safe Updates**: Version-tracked updates preserve customizations
3. **No Assumptions**: Works with any project structure
4. **Gradual Adoption**: Strict mode for new projects, relaxed for existing
5. **Developer Autonomy**: Enforce standards, enable customization

## Contributing

Contributions welcome! Please:

1. Follow the existing structure
2. Test with both new and existing projects
3. Document changes in README
4. Update version in `copier.yml`

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Related Tools

- [Copier](https://copier.readthedocs.io/) - Template management
- [Ruff](https://docs.astral.sh/ruff/) - Fast Python linter
- [Mypy](https://mypy-lang.org/) - Static type checker
- [pytest](https://pytest.org/) - Testing framework
- [pre-commit](https://pre-commit.com/) - Git hook framework

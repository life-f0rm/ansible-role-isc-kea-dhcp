# Contributing

## Development Setup

1. Create and activate a virtual environment.
2. Install development dependencies.
3. Run linting and Molecule tests before opening a pull request.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements-dev.txt
ansible-lint .
yamllint .
molecule test
```

## Commit Style

This repository uses Conventional Commits:

- `feat:` for new functionality
- `fix:` for bug fixes
- `docs:` for documentation changes
- `chore:` for maintenance tasks

Release automation uses these commit messages to determine version bumps.

## Security Expectations

- Do not add credentials, API keys, or private data to commits.
- Keep workflow permissions minimal.
- Prefer pinned dependencies and immutable GitHub Action SHAs.

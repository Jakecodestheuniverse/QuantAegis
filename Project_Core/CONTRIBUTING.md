# Contributing to [PROJECT_NAME]

Thank you for your interest in contributing. This project targets production use in regulated financial environments — that means we hold contributions to a high standard of quality, clarity, and safety. Please read this guide before opening a PR.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Who Should Contribute](#who-should-contribute)
3. [Getting Started](#getting-started)
4. [Development Workflow](#development-workflow)
5. [Contribution Types](#contribution-types)
6. [Code Standards](#code-standards)
7. [Testing Requirements](#testing-requirements)
8. [Documentation Standards](#documentation-standards)
9. [Regulatory & Compliance Considerations](#regulatory--compliance-considerations)
10. [Review Process](#review-process)
11. [Recognizing Contributors](#recognizing-contributors)

---

## Code of Conduct

All contributors are expected to follow our [Code of Conduct](CODE_OF_CONDUCT.md). We take this seriously. Financial AI has real-world stakes.

---

## Who Should Contribute

We actively welcome contributors with backgrounds in:

- **Quantitative Finance** — risk models, factor models, derivatives pricing
- **AI/ML Engineering** — LLMs, RAG, fine-tuning, multi-agent systems
- **Financial Compliance & Regulation** — MiFID II, SEC/FINRA, Basel III, DORA
- **Data Engineering** — financial data pipelines, market data connectors
- **Security Engineering** — especially financial-grade security practices
- **Technical Writing** — documentation, tutorials, examples

---

## Getting Started

### 1. Fork & Clone

```bash
git clone https://github.com/YOUR_USERNAME/[PROJECT_NAME].git
cd [PROJECT_NAME]
git remote add upstream https://github.com/[ORIGINAL_ORG]/[PROJECT_NAME].git
```

### 2. Set Up Environment

```bash
python -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
pre-commit install
```

### 3. Create a Branch

```bash
git checkout -b feature/your-descriptive-branch-name
```

Branch naming convention:
- `feature/` — new functionality
- `fix/` — bug fixes
- `docs/` — documentation only
- `compliance/` — regulatory rule updates
- `perf/` — performance improvements
- `refactor/` — code restructuring

---

## Development Workflow

1. **Check Issues first** — look for `good first issue` or `help wanted` labels
2. **Open an Issue** before starting large features — discuss the approach first
3. **Keep PRs focused** — one logical change per PR
4. **Write tests** — all new code must have tests (see below)
5. **Update docs** — if your change affects behavior, update the docs
6. **Self-review** — review your own diff before requesting review

---

## Contribution Types

### Bug Reports

Use the GitHub Issues bug template. Include:
- Minimal reproducible example
- Expected vs. actual behavior
- Environment details (Python version, OS, dependencies)
- Any relevant logs or stack traces

### Feature Requests

Open a GitHub Discussion under "Ideas" before filing a feature issue. Large features should go through an RFC (Request for Comments) process — see `docs/rfcs/`.

### Pull Requests

- Keep diffs under ~400 lines where possible
- Reference the issue being addressed (`Closes #123`)
- Ensure CI passes before requesting review
- Add your change to `CHANGELOG.md` under `[Unreleased]`

---

## Code Standards

We use strict linting. All of this is enforced automatically via pre-commit hooks.

| Tool | Purpose |
|---|---|
| `ruff` | Fast Python linting + formatting |
| `mypy` | Static type checking (strict mode) |
| `bandit` | Security vulnerability scanning |
| `pre-commit` | Runs all checks before commit |

```bash
# Run all checks manually
pre-commit run --all-files

# Type check
mypy src/

# Security scan
bandit -r src/ -ll
```

**Key rules:**
- All functions must have type annotations
- No `Any` types without explicit justification comment
- No hardcoded credentials, API keys, or secrets — ever
- No external network calls in unit tests
- Financial calculations must use `decimal.Decimal`, never `float`

---

## Testing Requirements

All contributions must include tests. We target ≥ 90% coverage on new code.

```bash
# Run the full test suite
pytest tests/ -v --cov=src --cov-report=term-missing

# Run only unit tests (fast)
pytest tests/unit/ -v

# Run integration tests (requires credentials in .env)
pytest tests/integration/ -v -m integration
```

**Test categories:**
- `tests/unit/` — pure logic, no I/O, must run in < 5s total
- `tests/integration/` — external services, marked `@pytest.mark.integration`
- `tests/compliance/` — regression tests for regulatory rule correctness

**Critical:** Any change to risk calculation logic, compliance rule evaluation, or audit trail behavior requires **two independent reviewers** before merge.

---

## Documentation Standards

- All public functions/classes need Google-style docstrings
- New modules need a module-level docstring explaining purpose
- User-facing features need an entry in `docs/`
- Complex financial logic needs inline comments explaining the *why*, not just the *what*

---

## Regulatory & Compliance Considerations

This project is used in regulated environments. Contributors must:

- **Never include real financial data** (even anonymized) in test fixtures — use synthetic data only
- **Document regulatory basis** for any compliance rule implementation (cite the specific regulation/article)
- **Flag breaking changes** to audit trail schemas explicitly — these require a deprecation window
- **Avoid model outputs that could be construed as financial advice** — all outputs must be clearly labeled as analytical tools

If you're unsure whether something has regulatory implications, open a Discussion and ask.

---

## Review Process

1. **Automated CI** must pass — no exceptions
2. **One maintainer approval** required for all PRs
3. **Two maintainer approvals** required for:
   - Risk calculation changes
   - Audit/compliance module changes
   - Security-sensitive changes
   - Breaking API changes
4. Maintainers aim to review within **5 business days**

---

## Recognizing Contributors

All contributors are listed in `CONTRIBUTORS.md`. Significant contributors may be invited to join the maintainer team. We follow the [All Contributors](https://allcontributors.org/) specification.

---

*Questions? Open a GitHub Discussion — we're friendly.*

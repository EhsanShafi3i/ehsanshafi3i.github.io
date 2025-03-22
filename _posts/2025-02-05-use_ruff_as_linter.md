---
title: 🚀 Boost Your Code Quality with pre-commit & Ruff – The Ultimate Guide!
image:
   path: /assets/posts/ruff.jpg
comment: true
tags: [python,linter,ruff]
---

## Why Use a Linter & Formatter?

Writing clean, consistent, and error-free code is crucial, but manually enforcing style rules is tedious. That’s where linters and formatters come in:

* 🔍 **Linters** (like Flake8, Pylint, Ruff) find errors, bad practices, and code smells before they cause issues.

* 🎨 **Formatters** (like Black, Ruff) automatically fix code style to keep everything consistent.

By using them, you:

* ✅ Catch bugs early before they break production

* ✅ Maintain a clean & readable codebase

* ✅ Ensure consistency across your team

* ✅ Save time by automating style enforcement

## Tools We Will Use: pre-commit and Ruff

Using automated tools for code checking and formatting helps teams improve code quality and prevent common issues. In this section, we introduce pre-commit and Ruff.

## Why Use pre-commit?

pre-commit is a local CI tool that runs before committing your code—acting as the first line of defense before GitHub Actions or other CI/CD pipelines.

* ✅ Catches issues early, before they reach the repo

* ✅ Runs faster than server-side CI (like GitHub Actions) since it works locally

* ✅ Ensures code quality before pushing

## Why Use Ruff Instead of Other Linters & Formatters?

Ruff is a blazing-fast Python linter, formatter, and code analyzer, designed to replace tools like Flake8, Black, isort, pylint, and more. Here’s why it stands out:

* 🔥 **Super Fast** — Written in Rust, it’s 10–100x faster than traditional Python linters

* 📦 **All-in-One** — Combines linting, formatting, and import sorting in a single tool

* 🚀 **Lightweight** — Minimal dependencies, making it perfect for large projects

* 🛠 **Highly Configurable** — Supports fine-tuned rule selection and customization

## How to Configure pre-commit & Ruff

## 1️⃣ Install Packages
```bash
    pip install pre-commit ruff
```
## 2️⃣ Configure pre-commit

Create a file in your git repo called .pre-commit-config.yaml
```dockerfile
    repos:
      - repo: https://github.com/astral-sh/ruff-pre-commit
        rev: v0.3.4  # Use the latest version
        hooks:
          - id: ruff
            args: [--fix]  # Be careful, this argument will fix some of the problems
          - id: ruff-format
```
## 3️⃣ Configuring ruff

Create a configuration file to define various rules for checking and formatting your code.
```toml
    # pyproject.toml
    [tool.ruff]
    line-length = 88  # Sets the max line length (default is 88, same as Black)
    target-version = "py312"  # Specifies the Python version (py312 for Python 3.12)
    extend-select = [
      "E",  # Style errors (PEP8)
      "F",  # Used variables and syntax errors
      "I",  # Import sorting
      "B",  # Common bugs
      "C",  # Complexity
      "N",  # Naming
      "UP", # Python upgrades
      "S",  # Security
      "RET", # Return statements
      "D"  # Docstrings (Ensures proper documentation)
    ]
    exclude = ["/**/__init__.py"] # Files that you want to ignore
```
Additional options:
```toml
    ignore = ["D100", "D104", "E501"]
    [tool.ruff.per-file-ignores]
    "tests/*" = ["S101"]  # Ignore security warnings in test files
    fixable = ["E", "F", "I", "B"]  # You can also disable auto-fixing for specific rules
    [tool.ruff.isort]
    known-first-party = ["my_project"]  # Treat "my_project" as first-party
    combine-as-imports = true  # Merge "import X as Y" statements
    [tool.ruff.pydocstyle]
    convention = "google"  # Options: "google", "numpy", "pep257"
```
## 4️⃣ Install pre-commit Hooks
```bash
    pre-commit install
```
## 5️⃣ Running pre-commit and Checking Code with Ruff 🔥🔥🔥

After installation and configuration, you can run pre-commit to check all project files. Running this command may produce an output similar to:
```bash
    Check ruff..............................................................Passed
    Check ruff-format......................................................Passed
```
If any errors exist, pre-commit will report them, and in some cases, Ruff will automatically fix the issues.

## References

For more details, check out these resources:

* [Ruff Documentation](https://beta.ruff.rs/docs/)

* [Pre-commit Documentation](https://pre-commit.com/)

* [GitHub Ruff Repository](https://github.com/astral-sh/ruff)



This plan is designed to publish `igrep` safely with a test cycle first, then production release.

  

## 1. Packaging Readiness

  

Before first publish, confirm:

  

- `pyproject.toml` has correct metadata (`name`, `version`, `description`, `authors`, `license`, `readme`, `classifiers`).

- `README.md` renders properly on PyPI.

- CLI entry point is present:

  

```toml

[project.scripts]

igrep = "src.cli:main"

```

  

- License file exists in repository root.

- Version follows semantic versioning (`0.1.0`, `0.1.1`, `0.2.0`, etc.).

  

## 2. Build and Validate Locally

  

From repository root:

  

```powershell

uv sync

uv build

```

  

This should generate:

  

- `dist/igrep-<version>.tar.gz`

- `dist/igrep-<version>-py3-none-any.whl`

  

Validate metadata and long description:

  

```powershell

uvx twine check dist/*

```

  

## 3. Test Publish to TestPyPI

  

Create an account on TestPyPI and upload there first:

  

```powershell

uvx twine upload --repository testpypi dist/*

```

  

Install from TestPyPI in a clean virtual environment:

  

```powershell

pip install -i https://test.pypi.org/simple/ igrep==<version>

igrep --help

```

  

Smoke-test:

  

1. `igrep setup`

2. `igrep --track "<folder>"`

3. `igrep sync`

4. `igrep "sample text"`

  

## 4. Production Publish to PyPI

  

After TestPyPI verification:

  

```powershell

uvx twine upload dist/*

```

  

Verify package page and install:

  

```powershell

pip install igrep==<version>

igrep --help

```

  

## 5. Recommended CI/CD (Trusted Publishing)

  

Use GitHub Actions with PyPI Trusted Publishing (OIDC) so no API token is stored in secrets.

  

High-level workflow:

  

1. Trigger on version tags (example: `v0.1.1`).

2. Build artifacts with `uv build`.

3. Run `twine check`.

4. Publish with `pypa/gh-action-pypi-publish`.

  

Suggested release flow:

  

1. Update version in `pyproject.toml`.

2. Commit changes.

3. Tag release (`git tag vX.Y.Z`).

4. Push commit and tag.

5. CI publishes to PyPI.

  

## 6. Release Checklist (Each Version)

  

- Update version in `pyproject.toml`.

- Update changelog/release notes.

- Run local tests and CLI smoke tests.

- Build artifacts (`uv build`).

- Validate artifacts (`uvx twine check dist/*`).

- Publish to TestPyPI (recommended).

- Publish to PyPI.

- Create GitHub Release notes.

  

## 7. Notes Specific to igrep

  

- `igrep` depends on external Tesseract OCR runtime. Mention this clearly in PyPI description and docs.

- Model download happens during `igrep setup`; this is expected behavior for first run.

- Current Python requirement is `>=3.13`. If broader adoption is required, test and lower this (for example to `>=3.10`) before public release.
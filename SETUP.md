# Project setup with uv and Poetry

This guide uses `uv` only to create and manage the Python virtual environment.
Poetry installs the project dependencies into that environment.

The default setup below is intended for both development and running `pytest`.
It installs the main project dependencies and the `tests` dependency group, which
includes packages such as `stim` and `qibojit`.

## Prerequisites

- Python managed by `uv`
- Poetry installed inside the project virtual environment using `uv`
- PowerShell on Windows

The project supports Python `>=3.10,<3.14`. Python 3.12 is recommended because
some optional dependencies are constrained below Python 3.13.

## 1. Open the project

```powershell
cd C:\Users\AMAL\Projects\my_qibo
```

## 2. Create the virtual environment with uv

```powershell
uv python install 3.12
uv venv venv --python 3.12
```

Activate it:

```powershell
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks activation for the current shell session:

```powershell
Set-ExecutionPolicy -Scope Process Bypass
.\venv\Scripts\Activate.ps1
```

Confirm that the active Python is the one inside this project:

```powershell
python -c "import sys; print(sys.executable)"
```

The printed path should start with:

```text
C:\Users\AMAL\Projects\my_qibo\venv\
```

## 3. Install Poetry inside the virtual environment

```powershell
uv pip install poetry
```

This uses `uv`'s package installer and does not call standalone `pip`.

Confirm Poetry is available:

```powershell
poetry --version
```

## 4. Tell Poetry to use the active virtual environment

Poetry normally creates its own virtual environment. Disable that behavior for
this shell session so Poetry installs into the `uv` environment that is already
active.

```powershell
$env:POETRY_VIRTUALENVS_CREATE = "false"
```

Optional sanity check:

```powershell
poetry config virtualenvs.create
```

For this workflow, the effective value should be `false`.

## 5. Install project dependencies

Use `main,tests` for the standard development and test environment:

```powershell
poetry install --sync --only main,tests
```

This installs the package in editable mode and includes the test dependencies,
including `pytest`, `stim`, `qibojit`, `matplotlib`, and related test packages.

Avoid installing the `dev` group unless you specifically need it. In this
project, the `dev` group includes `pdbpp`, and the locked dependency combination
can make pytest fail during startup on Windows with:

```text
AttributeError: module 'fancycompleter' has no attribute 'LazyVersion'
```

The `main,tests` install avoids that problem and is sufficient for running the
test suite.

## 6. Verify the installation

```powershell
poetry run python -c "import qibo, stim, qibojit; print('ok')"
poetry run pytest
```

To run one test file:

```powershell
poetry run pytest tests/test_ui_bloch_sphere.py
```

## Running local scripts

If you create a script such as `main.py` in the project root, run it through
Poetry so it uses the same environment and dependencies:

```powershell
poetry run python main.py
```

For example, if `main.py` contains:

```python
from qibo import Circuit, gates

circuit = Circuit(1)
circuit.add(gates.H(0))
print(circuit)
```

run:

```powershell
poetry run python main.py
```

You can also run the script directly after activating the virtual environment:

```powershell
python main.py
```

`uv run python main.py` can work, but it is not the recommended command for
this setup. This guide deliberately lets Poetry own dependency installation and
execution after the virtual environment is created. Using `poetry run` keeps the
workflow consistent with `poetry.lock` and avoids `uv` trying to resolve or sync
dependencies separately.

## Daily workflow

Each new PowerShell session only needs:

```powershell
cd C:\Users\AMAL\Projects\my_qibo
.\venv\Scripts\Activate.ps1
$env:POETRY_VIRTUALENVS_CREATE = "false"
poetry run pytest
```

## Optional dependency groups

Install documentation dependencies only when needed:

```powershell
poetry install --sync --only main,tests,docs
```

Install CUDA 11 dependencies only on a matching CUDA 11 environment:

```powershell
poetry install --sync --only main,tests,cuda11
```

Install CUDA 12 dependencies only on a matching CUDA 12 environment:

```powershell
poetry install --sync --only main,tests,cuda12
```

## If the wrong environment is used

Check which Python Poetry is using:

```powershell
poetry run python -c "import sys; print(sys.executable)"
```

If it does not point to `C:\Users\AMAL\Projects\my_qibo\venv\`, reactivate the
environment and set the Poetry environment variable again:

```powershell
.\venv\Scripts\Activate.ps1
$env:POETRY_VIRTUALENVS_CREATE = "false"
poetry install --sync --only main,tests
```

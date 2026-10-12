# Local development setup

This guide gets the current `exam-prep` Python project running locally. The repository is still a starter project: it currently has a Python command-line entry point, but does not yet include a frontend, FastAPI backend, AI integration, or MCP server. Those components will need their own run commands once they are added.

## Prerequisites

- Git
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- Python 3.14 or later. The project pins its Python version to 3.14 in `.python-version`; uv can install and select it for you.
- macOS: Terminal, iTerm, or another shell.
- Windows: PowerShell 5.1 or later (PowerShell 7 recommended).

### Install uv

#### macOS

With Homebrew:

```sh
brew update
brew install uv
```

If uv is already installed with Homebrew:

```sh
brew update
brew upgrade uv
```

Alternatively, use uv's standalone installer:

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### Windows

In PowerShell, install uv with WinGet:

```powershell
winget install --id=astral-sh.uv -e
```

Alternatively, use uv's standalone installer in PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

If you use another platform or package manager, follow uv's official [installation instructions](https://docs.astral.sh/uv/getting-started/installation/).

After installation, open a new terminal window and confirm:

```sh
uv --version
```

## Get the project

Clone the repository using the URL provided by your team, then change into the project directory.

#### macOS (Terminal)

```sh
git clone <REPOSITORY_URL>
cd exam-prep
```

#### Windows (PowerShell)

```powershell
git clone <REPOSITORY_URL>
Set-Location exam-prep
```

If you already have the repository, open a terminal in its `exam-prep` directory instead.

## Set up your environment

Create a local `.env` file from the sample.

#### macOS (Terminal)

```sh
cp .env-sample .env
```

#### Windows (PowerShell)

```powershell
Copy-Item .env-sample .env
```

Edit `.env` and replace the example values with credentials you are authorized to use. Leave credentials blank for integrations you are not using. **Never commit `.env` or share real API keys, AWS access keys, or session tokens.** The `.env-sample` file is safe to share because its values are placeholders.

The sample includes:

- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and `GOOGLE_API_KEY` for possible LLM providers.
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, and `AWS_REGION` for AWS.

Use only the credentials required by the integration you are working on. For AWS development, prefer your organization's approved AWS IAM Identity Center/SSO or named profile workflow where available rather than creating long-lived access keys.

The starter project does not currently load `.env` or call any LLM or AWS service. Adding values alone will not activate those integrations.

## Install project dependencies

From the `exam-prep` directory, run this command on either macOS or Windows:

```sh
uv sync
```

uv creates a local `.venv`, installs the project dependencies, and creates or updates `uv.lock` as needed. Keep `uv.lock` under version control so the team can reproduce dependency versions.

## Run the current project

Run its current command-line entry point on either macOS or Windows:

```sh
uv run exam-prep
```

At this starter stage, successful output is:

```text
Hello from exam-prep!
```

To run the command again after changing Python source, rerun `uv run exam-prep`; uv uses the project's environment.

## Updating dependencies

After pulling changes that update `pyproject.toml` or `uv.lock`, synchronize your local environment on either macOS or Windows:

```sh
uv sync
```

To intentionally update dependencies and the lockfile, run:

```sh
uv lock --upgrade
uv sync
```

Review and commit the resulting `uv.lock` changes with the dependency changes.

## Planned application components

The frontend, FastAPI API, AI integration (LangChain or AWS Strands), and MCP server have not been added yet. Their framework choices, configuration, and startup commands should be documented here when they exist. Do not expect frontend, API, or MCP server commands to work in the current starter project.

## Troubleshooting

- **`uv: command not found` (macOS):** install uv using the instructions above, open a new terminal, and check `uv --version`. If installed with the standalone script, ensure uv's install directory is on your `PATH`.
- **`uv` is not recognized (Windows):** install uv using the instructions above, open a new PowerShell window, and check `uv --version`. If it is still not recognized, confirm the install directory is on your `PATH`.
- **Python version issue:** run `uv python install 3.14`, then retry `uv sync`.
- **Command not found for `exam-prep`:** confirm you are in the directory containing `pyproject.toml`, then run `uv sync` and retry `uv run exam-prep`.
- **Missing dependency or import error:** run `uv sync` to synchronize the environment with the project definition and lockfile.
- **Credentials do not appear in the application:** `.env` is not loaded by the current starter project. The application must explicitly load or otherwise receive environment variables when integrations are implemented.

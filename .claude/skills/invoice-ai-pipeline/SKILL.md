---
name: invoice-ai-pipeline
description: Set up the invoice PDF extraction microservice (Python, FastAPI, Claude API) and the GitHub Actions pipelines for Claude PR review and bug auto-fix in the current folder, by copying a tested reference project. Takes one argument - service, pipelines or all.
argument-hint: service | pipelines | all
disable-model-invocation: true
---

# invoice-ai-pipeline

Sets up an invoice PDF extraction microservice (Python, FastAPI, Claude API) and the GitHub Actions pipelines for Claude pull request review and bug auto-fix in the **current folder**, by copying a tested reference project.

## Argument

One argument, required: `service`, `pipelines` or `all`. If it is missing or any other value, stop and ask the user to choose one of the three. Do not guess.

- `service` runs Stage service only.
- `pipelines` runs Stage pipelines only.
- `all` runs Stage service, then Stage pipelines.

## Reference project

- Repository: https://github.com/mahesh-mohite-ESL/invoice-extractor-ai-ci-pipeline-demo
- Commit: `9c1a418`

## Rules (apply to every stage)

1. **Copy files exactly.** Never rewrite, fix, reformat or improve any copied file. The service contains **five deliberate bugs** that the auto-fix pipeline repairs later. "Fixing" them defeats the purpose of the demo. If a test fails because of them, report it; do not patch the code.
2. **Never overwrite an existing file that differs** from the reference copy without asking the user first. Identical files can be skipped silently. Missing files are created. List the differing files and ask what to do (skip, overwrite, or stop).
3. **Never run `git init`, `git commit` or `git push`.** Do not run any other git command in the project folder either. Git is used only inside the temporary clone.
4. **Never ask for an API key or token.** Do not create `.env` files with secrets. Tell the user in the summary that secrets are configured later (see docs/SETUP-GUIDE.md after Stage pipelines).
5. **End every stage with a summary** of what was created (and what was skipped), the test result, and the named next step.

## Fetching the reference

Do this once per run, before the first stage that needs it.

1. Create a temporary folder **outside the project** (for example under the system temp directory or the session scratchpad) with a unique name, for example `invoice-ref-<random>`. It must not be inside the current folder.
2. `git clone https://github.com/mahesh-mohite-ESL/invoice-extractor-ai-ci-pipeline-demo <tmp>/ref`
3. `git -C <tmp>/ref checkout 9c1a418`
4. Verify `git -C <tmp>/ref rev-parse --short HEAD` prints `9c1a418`. If the clone or checkout fails, stop and report the error; do not fall back to another source.
5. Copy entries from `<tmp>/ref` into the current folder as listed per stage, preserving relative paths. Copy directories recursively. Do not copy the `.git` folder.
6. When all requested stages have copied their files, **delete the temporary folder** (also delete it if the run aborts after cloning). Confirm it is gone. Never delete anything inside the project folder as part of cleanup.

On Windows use PowerShell-compatible commands (`Copy-Item -Recurse`, `Remove-Item -Recurse -Force`) or the Bash tool, whichever is available. Run the git commands for the clone with the temp folder as the target path, never inside the project.

## Stage service

Copy these entries from the reference:

- `app`
- `samples`
- `tests`
- `tools`
- `pyproject.toml`
- `requirements.txt`
- `requirements-dev.txt`
- `Dockerfile`
- `.dockerignore`
- `.gitignore`
- `.gitattributes`
- `README.md`
- `docs/PLANTED-BUGS.md` (create `docs/` if needed; copy only this file, since other files in docs belong to Stage pipelines)

Then:

1. Create the virtual environment: `python -m venv .venv` (use `py -3 -m venv .venv` on Windows if `python` is unavailable). If Python is not installed, stop the stage after copying, say so, and name installing Python 3.11+ as the next step.
2. Install into it: `.venv\Scripts\python -m pip install -r requirements-dev.txt` (Windows) or `.venv/bin/python -m pip install -r requirements-dev.txt` (macOS/Linux).
3. Run `pytest` with the venv interpreter: `.venv\Scripts\python -m pytest`.
4. **Expected result: 28 passed.** Report the actual result. If it differs, show the failing output and do not edit any file to make it pass.

Summary and next step: list what was created, the pytest result, and the next step. Next step is `/invoice-ai-pipeline pipelines` if only `service` was run, or the Stage pipelines step if running `all`.

## Stage pipelines

**Precondition:** `app/main.py` must exist in the current folder. If it does not, stop this stage, explain that the service is required first, and name `/invoice-ai-pipeline service` as the next step. Do not copy anything.

Copy these entries from the reference:

- `.github`
- `scripts`
- `.claude`
- `package.json`
- `CLAUDE.md`
- `docs/flawed-change.patch`
- `docs/SETUP-GUIDE.md`

Note: `.claude` in the reference may overlap with this skill's own folder. Never overwrite or delete `.claude/skills/invoice-ai-pipeline/SKILL.md`; skip that path if it appears in the reference. Apply Rule 2 to everything else.

Then, only if Node.js is installed (check `node --version`):

1. Run `npm run test:autofix`.
2. **Expected result: 12 passing.** Report the actual result. Do not edit files to make it pass.

If Node.js is not installed, skip the test, say so, and name installing Node.js as a follow-up.

Summary and next step: list what was created, the test result (or that it was skipped), and the next step. The next step is to read `docs/SETUP-GUIDE.md` and configure the repository secrets and GitHub settings it describes. The user does this themselves; the skill never asks for or handles credentials. Mention that git init, commit and push are left to the user.

## Stage all

Run Stage service, then Stage pipelines, using one clone of the reference. If Stage service stops early (for example Python missing), still attempt Stage pipelines only when `app/main.py` exists. Give one combined final summary.

## Do not

- Do not modify, lint or format copied files.
- Do not run `git init`, `git commit` or `git push`.
- Do not request, read or store an API key or token.
- Do not overwrite differing existing files without asking.
- Do not leave the temporary clone behind.

# GitHub Actions

GitHub Actions is GitHub's built-in automation platform. In ML projects, you can use it to run tests before merging code, check formatting, build containers, pull versioned data, or trigger model-training jobs.

In this first lesson, you will create a small pull request workflow. The example is intentionally simple: start with a Python function and a pytest check, then require that check before code can merge into `main`.

## Workflow

```mermaid
flowchart TD
    A["Open pull request"] --> B["GitHub Actions<br>starts workflow"]
    B --> C["Install Python<br>dependencies"]
    C --> D["Run pytest"]
    D --> E{"Tests pass?"}
    E -->|Yes| F["Review and merge"]
    E -->|No| G["Fix branch and push again"]
```

## Create a test workflow

1. Create a **new, empty repository** on GitHub for this exercise. It will be used in lesson 1 only. Keep it separate from the ML repository you created from this template, which will be used in lessons 2 to 5.

2. Clone the repository, `cd` into it, and set it up as a uv project with Python 3.13 and pytest:

   ```bash
   echo "3.13" > .python-version
   uv init --bare
   uv add --dev pytest
   ```

   This creates `pyproject.toml` and `uv.lock`. Commit both, together with `.python-version`, so GitHub Actions installs exactly the same environment.

3. Create `src/calculator.py`:

   ```python
   def add(a, b):
       return a + b
   ```

4. Create `tests/test_calculator.py`:

   ```python
   from src.calculator import add


   def test_add():
       assert add(1, 2) == 3
       assert add(-2, 2) == 0
       assert add(0, 0) == 0
   ```

5. Create `.github/workflows/test.yaml`:

   ```yaml
   name: Tests

   on:
     pull_request:
       branches:
         - main

   permissions:
     contents: read

   jobs:
     test:
       runs-on: ubuntu-latest
       timeout-minutes: 10
       steps:
         - uses: actions/checkout@v6

         - name: Set up uv
           uses: astral-sh/setup-uv@v7

         - name: Install dependencies
           run: uv sync --locked

         - name: Run tests
           run: uv run python -m pytest
   ```

6. Push these starter files to `main`.

7. Go to **Settings > Branches > Add classic branch protection rule** and set:

   - Branch name pattern: `main`
   - Require a pull request before merging: **Check**
   - Require approvals: **Check**
   - Require status checks to pass before merging: **Check**

   Then click on **Create**.

![GitHub branch protection rule settings](./assets/01-protection-rule.png)

8. Create a new branch, change the calculator or test, commit, and open a pull request into `main`. You should see the workflow start automatically.

![Pull request status check running in GitHub](./assets/01-pr-check.png)

## Interpret the result

The pull request check is a feedback gate. A passing check means the repository can recreate the test environment and run the test suite from scratch. A failing check means the branch needs another commit before it is ready to merge.

The starter workflow only asks for `contents: read` because it only needs to check out the repository and run tests. Later, the CML workflow asks for write permissions because it needs to publish a pull request comment.

`uv sync --locked` reads `.python-version`, installs Python 3.13, and installs the exact versions pinned in `uv.lock`. The `--locked` flag makes the job fail if `uv.lock` is out of date with `pyproject.toml`, so a forgotten lockfile update shows up in the pull request instead of silently changing versions.

The `timeout-minutes` setting is a small safety guard: if a dependency install or test run gets stuck, GitHub stops the job instead of letting it run until the repository limit is reached.

In a larger ML repo, this same pattern can run data validation, linting, model smoke tests, or training jobs. Later lessons extend this workflow with DVC and CML.

## Exercise

- Add a `subtract(a, b)` function and a matching pytest test.
- Push the branch and confirm the workflow runs again.
- Break one assertion on purpose, observe the failure, then fix it with a new commit.

## Return to the ML repository

Lessons 2 to 5 run in the ML repository you created from this template, not in the calculator repository. Before continuing, move back into it:

```bash
cd <path-to-ml-repository>
```

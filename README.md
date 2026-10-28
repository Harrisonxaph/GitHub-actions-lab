# GitHub Actions lab

A tiny calculator for learning continuous integration (CI): automatically running
checks whenever code changes. It uses Python 3.13 and its built-in `unittest`
module. There are no packages to install.

## What each file does

| File | Purpose |
| --- | --- |
| `calculator.py` | Defines `add`, `subtract`, `multiply`, and `divide`. Dividing by zero raises Python's `ZeroDivisionError`. |
| `test_calculator.py` | Checks all four operations, including negative numbers, zero, fractional division, and division by zero. |
| `.github/workflows/python-tests.yml` | Tells GitHub when and how to run the tests. |
| `README.md` | Explains the project and the practice exercise below. |
| `LICENSE` | Contains the repository's existing license terms. |

## Run the tests locally

Install Python 3.13, open a terminal in this repository, and run:

```sh
python -B -m unittest discover -v
```

If your system calls Python `python3.13`, use that instead of `python`.
`-B` avoids creating bytecode cache files. `discover` finds files named
`test*.py`, and `-v` prints each test's name. You should see five tests and `OK`.
`assertEqual` compares actual and expected results; `assertRaises` verifies that
division by zero raises the expected exception. A failed check makes the command
exit with a nonzero status, which also tells GitHub Actions the job failed.

## How GitHub Actions works

A **workflow** is an automation described in a YAML file under
`.github/workflows/`. An **event** starts a workflow run. This workflow listens
to pushes on every branch and pull request events (including opening a PR and
pushing new commits to it). A push to an open PR can therefore produce two runs.

The workflow contains one **job**, `test`, running on `ubuntu-latest`, a fresh
Linux virtual machine hosted by GitHub. Its **steps** run in order:

1. **Check out the repository** uses the official `actions/checkout@v7` action
   to copy the code onto the runner. `persist-credentials: false` keeps Git
   credentials out of the checkout's configuration after checkout finishes.
2. **Set up Python 3.13** uses the official `actions/setup-python@v7` action to
   select Python 3.13 for the following command.
3. **Run unit tests** runs the same standard-library command used locally.
   There is no dependency installation step because none is needed.

`permissions: contents: read` gives the automatic `GITHUB_TOKEN` read-only access
to repository contents and leaves other permissions disabled. You do not need to
create a token or configure any secrets. This project uses no paid services;
standard GitHub-hosted runners are free for this public repository. For private
repositories, check your plan's Actions allowance before enabling runs.

## View test results on GitHub

1. Open the repository's **Actions** tab and select **Python tests**.
2. Open a run, select the **test** job, and expand **Run unit tests**.
3. Read the individual test names and the final `OK` or failure traceback.

You can also open a pull request's **Checks** tab. Green means the tests passed;
red means a check failed. A setup failure can also make a run red, so inspect
the failing step's log. If workflows are disabled, enable Actions in repository
settings first. This workflow does not merge code or enforce branch protection.

## Safely practice failing and fixing a test

Start with a clean working tree (`git status --short` should print nothing).
While this project's PR is still open, branch from `feature/first-ci`:

```sh
git switch feature/first-ci
git switch -c practice/failing-test
```

After the project has been merged by a maintainer, you can instead start from
an updated `main`, then create the same separate practice branch. Do not edit
or push directly to `main` for this exercise.

1. In `test_calculator.py`, change only `self.assertEqual(add(2, 3), 5)` to
   `self.assertEqual(add(2, 3), 6)`. This deliberately makes the expected answer
   wrong; the calculator should still add correctly.
2. Run the local test command. Expect `test_add` to fail with `5 != 6`.
3. Commit and push only the practice branch:

   ```sh
   git add test_calculator.py
   git commit -m "Practice a failing test"
   git push -u origin practice/failing-test
   ```

4. Open **Actions** and inspect the red run for `practice/failing-test`.
5. Change the expected answer back to `5` and rerun the tests. Expect `OK`.
6. Commit and push the fix:

   ```sh
   git add test_calculator.py
   git commit -m "Fix the practice test"
   git push
   ```

7. Inspect the new green run. Keep this exercise on its separate branch; no
   practice PR or merge is required. Return with `git switch feature/first-ci`
   (or `git switch main` if working after the project was merged).

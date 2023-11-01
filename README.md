# Badges

This is a repo for storing badges that we can link to in our README files.

## How can I add a badge to my repo?

You can create lots of different badges, but so far I've only experimented with Pylint, so this guide will focus on that. You can adapt this to other badges though!

### 1. Create a workflow

Create a file called `./github/workflows/run-pylint.yml`, and put the following inside (being sure to replace `<YOUR REPO>` with the name of your project, such as "puggle":

name: Pylint

  on:
    workflow_dispatch:
    push:
      branches:
        - main
    pull_request:
  
  jobs:
    build:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          python-version: ["3.11"] # Or whatever version of Python your code uses
      steps:
        - uses: actions/checkout@v3
        - name: Set up Python ${{ matrix.python-version }}
          uses: actions/setup-python@v3
          with:
            python-version: ${{ matrix.python-version }}
        - name: Install dependencies
          run: |
            python -m pip install --upgrade pip
            pip install pylint anybadge
        - name: Run Pylint and create badge
          run: |
            pylint $(git ls-files '*.py') --output pylint.log --exit-zero
        - name: Get Pylint score and create badge
          run: |
            PYLINT_SCORE=$(sed -n 's/^Your code has been rated at \([-0-9.]*\)\/.*/\1/p' pylint.log)
            echo $PYLINT_SCORE
            anybadge -l pylint --value=$PYLINT_SCORE -f <YOUR REPO>-pylint-badge.svg 2=red 4=orange 8=yellow 10=green
        - name: Push badge to badges repo
          uses: dmnemec/copy_file_to_another_repo_action@main
          env:
            API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
          with:
            source_file: "<YOUR REPO>-pylint-badge.svg"
            destination_repo: "nlp-tlp/badges"
            user_email: "github-actions[bot]@users.noreply.github.com"
            user_name: "github-actions[bot]"
            commit_message: "Update <YOUR REPO> Pylint badge"

This will create a GitHub action that runs Pylint, creates a corresponding badge, then pushes it to this repository.

### 2. Create a token and put it in your repo

In GitHub, click on your profile at the top right, then Settings -> Developer Settings -> Personal Access Tokens -> Tokens (classic). Generate a new token. Call it whatever you like, and tick all of the `repo` permissions. Copy the newly created token to the clipboard.

Then, go to your repository, then Settings -> Secrets and Variables -> Actions. Click "New Repository Secret". Set `API_TOKEN_GITHUB` as the name, and paste in the token you created earlier as the "Secret".

### 3. Create a link to the badge in your README file

In your readme file, paste the following:

    [![Pylint Status](https://github.com/nlp-tlp/badges/blob/main/<YOUR REPO>-pylint-badge.svg)](https://github.com/nlp-tlp/<YOUR REPO>/actions/workflows/run-pylint.yml)

... and then next time you push to main, you should have a nice Pylint badge!

If you have any issues with the above, please let me (Michael) know.

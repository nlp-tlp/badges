# Badges

This is a repo for storing badges that we can link to in our README files.

## How can I add a badge to my repo?

This repo provides instructions for how to create a badge to put on your repo's README, like this: [![Pylint Status](https://github.com/nlp-tlp/badges/blob/main/puggle-pylint-badge.svg)](https://github.com/nlp-tlp/puggle/actions/workflows/run-pylint.yml)

You can create lots of different badges, but so far I've only experimented with Pylint, so this guide will focus on that. You can adapt this to other badges though!

### 1. Create a workflow

The workflow below is designed for poetry-based projects, which typically have the following structure:

    my-package/
      my-file.py
      my-other-file.py
    tests/
    README.md

Your repo should have the same name as your poetry package, e.g. in this case `my-package`. If your code is not structured this way, you will need to modify the workflow below as necessary.

In your repo, create a file called `./github/workflows/run-pylint.yml`, and put the following workflow inside:

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
            python-version: ["3.11"]
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
          - name: Download .pylintrc file
            uses: suisei-cn/actions-download-file@v1.4.0
            id: pylintrc
            with:
              url: "https://raw.githubusercontent.com/nlp-tlp/badges/main/.pylintrc"
          - name: Run Pylint
            run: |
              pylint ${{ github.event.repository.name }} --output pylint.log --exit-zero
          - name: Get Pylint score and create badge
            run: |
              PYLINT_SCORE=$(sed -n 's/^Your code has been rated at \([-0-9.]*\)\/.*/\1/p' pylint.log)
              echo $PYLINT_SCORE
              anybadge -l pylint --value=$PYLINT_SCORE -f ${{ github.event.repository.name }}-pylint-badge.svg 2=red 4=orange 8=yellow 10=green
          - name: Push badge to badges repo
            uses: dmnemec/copy_file_to_another_repo_action@main
            env:
              API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
            with:
              source_file: "${{ github.event.repository.name }}-pylint-badge.svg"
              destination_repo: "nlp-tlp/badges"
              user_email: "github-actions[bot]@users.noreply.github.com"
              user_name: "github-actions[bot]"
              commit_message: "Update ${{ github.event.repository.name }} Pylint badge"

This will create a GitHub action that runs Pylint, creates a corresponding badge, then pushes it to this repository. The badge will be named according to your repo, e.g. `puggle-pylint-badge.svg`.

You can view the status of this action by going to "Actions" at the top of your repo in GitHub.

Note that your code is linted using the `.pylintrc` file located in this repository (which is Google's one, and also the same one used by CTMTDS).

### 2. Create a token and put it in your repo

In GitHub, click on your profile at the top right, then Settings -> Developer Settings -> Personal Access Tokens -> Tokens (classic). Generate a new token. Call it whatever you like, and tick all of the `repo` permissions. Copy the newly created token to the clipboard.

Then, go to your repository, then Settings -> Secrets and Variables -> Actions. Click "New Repository Secret". Set `API_TOKEN_GITHUB` as the name, and paste in the token you created earlier as the "Secret".

### 3. Create a link to the badge in your README file

In your readme file, paste the following (replace `<YOUR REPO>` with the name of your repo):

    [![Pylint Status](https://github.com/nlp-tlp/badges/blob/main/<YOUR REPO>-pylint-badge.svg)](https://github.com/nlp-tlp/<YOUR REPO>/actions/workflows/run-pylint.yml)

... and then next time you push to main, you should have a nice Pylint badge!

If you have any issues with the above, please let me (Michael) know.

## Other badges you can add

You can also set up a GitHub action to run your unit tests - see the [run-tests.yml](https://github.com/nlp-tlp/puggle/blob/main/.github/workflows/run-tests.yml) action in the Puggle repo for example.

Adding a badge to show that your tests pass is pretty straightforward, as GitHub stores an svg of a badge with the status of that action. You can add it to your README like this:

    [![Pytest Status](https://github.com/nlp-tlp/<YOUR REPO>/actions/workflows/run-tests.yml/badge.svg)](https://github.com/nlp-tlp/<YOUR REPO>/actions/workflows/run-tests.yml)

Another useful badge is for your unit test coverage. In the `run-tests` action above, I've used [Coveralls](https://coveralls.io) to track the coverage history. The Coveralls website provides me with a link to the badge, which you can add to your repo:

    [![Coverage Status](https://coveralls.io/repos/github/nlp-tlp/<YOUR REPO>/badge.svg?branch=main)](https://coveralls.io/github/nlp-tlp/<YOUR REPO>?branch=main)

You may need to create an account with Coveralls to get this to work, but I'm not 100% sure.

## Other options to look into

[Shields](https://shields.io/) looks good, but I haven't tried it yet. Might be better than Coveralls for the test coverage badge - feel free to give it a go if you like, and let me know how it goes.

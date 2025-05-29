# Introduction to Github Actions

## Learning Objectives

1. Use `git` in practice with Github
1. Get familiar with Github repos, Actions, Packages
1. Build and deploy a Python package from a Github repository using Github Actions

## Github

You should all have created a Github account (free) or be using an existing Github account. If not, please create one using the pre-read instructions and links.

Github is a hosted and managed service for git repositories, Actions (CICD workflows tool), Packages (artifact repository).

Note: Unfortunately, Github Packages doesn't support Python Packages (PyPI) so we will use a hosted PyPi repository in GitLab. You will not need accounts or anything as the credentials will be posted at the appropriate step in this guide. The registry will be shared between all students and colleagues so ensure you use unique package names. All packages will be deleted on 1st of each month so you may need to republish packages.

## 1. New Git repository

In the top right, click on "New" button and select repository.

![new repo](img/gh-new-repo.jpeg)

Enter a simple name for your repo like `cspark-cicd-lesson` (You all can reuse this name as it will be under your user account). Add a README and make the repository private.

![new repo page](img/gh-new-repo-page.jpeg)

This will create you a new Github repository with a README.md file with the first commit called "Initial commit".

![repo landing page](img/gh-repo-landing.jpeg)

### 1.1 IDE

Now you have three options to develop code:

1. Use the Github provided VS Code in the browser 
1. Use VS Code that you have installed locally on your machine
1. Any other code or file editor you have installed locally on your machine

#### Github provided VS Code

To open this, press the `.` key once on the repository page. This service is called github.dev. Alternatively you can change the URL for your repository from `github.com` to `github.dev`. For exmaple: My repository URL is `https://github.com/<username>/cspark-cicd-lesson`. I can open browser based IDE using `https://github.dev/<username>/cspark-cicd-lesson`. In both cases it will open it as such:

![github.dev](img/github-dev.jpg)

#### Local VS Code

This lesson will show you the instructions using the locally installed VS Code

First clone the repository locally using the HTTPS method. Click on "Code" -> HTTPS -> Copy the URL. 

![code url](img/gh-code-url.jpeg)

In a terminal run `git clone <url>`. This will clone the git repository locally.

![git clone](img/git-clone.jpg)

Now open VS Code either by running the command `code` in the cloned directory or from your Start menu and click Open folder.

![vscode open](img/vscode-open.jpg)

This will open the directory, ready to edit.

### 1.2 Git Branch

The first thing we always do when working with git repositories is change to a new branch. In your terminal create and switch to a new branch as follows

```bash
git checkout -b lesson
```

## 2. Github Actions (Workflows)

[Github Actions](https://docs.github.com/en/actions) is a service feature that allows executing jobs and pipelines for the purpose of Continuous Integration and Delivery. The workflows are defined using YAML language and has a special [schema and syntax](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions) that we will reference all through this lesson.

### 2.1 Concepts

#### Workflows

Workflows are a combination of steps equivalent to a pipeline. There can be multiple workflows contained within a repository. For Github Actions to automatically discover your workflows, they must be stored a `yml` or `yaml` extension files within the `.github/workflows/` directory of your repository. 

Workflows consist of triggers and jobs. Triggers define events that start the workflow. These could be a commit or a push or another workflow completing or a cron based schedule to run regularly. 

#### Jobs

Jobs are a set of steps to execute. This can be one or more shell commands or scripts to execute or it could an Action (more below). These jobs are executed on "Runners". Runners are the agents that help run any commands for your workflows. Each job runs on a different agent thus the ways data can be passed between jobs is limited.

#### Actions

Actions are pre-built reusable script that only need some configuration to be added. These can perform complex actions and are usually developed in Javascript language.


## 3. Create Hello World Workflow

Let's start building up the workflow step by step. The simplest workflow we will create will be to echo "Hello World".

Create a new folder and file under `.github/workflows/helloworld.yaml` and add the following content:


```yaml
name: Hello World GitHub Action

on: [push, workflow_dispatch]
jobs:
  Hello-World:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello World from Github Actions"
```
Let's breakdown the definition.

- `name` is simply any name you want to give the workflow. 
- `on` defines the triggers. It is a list or a single trigger. 
  - `push` trigger references the git push action. 
  - `workflow_dispatch` is to enable manual trigger action from the UI. There are other triggers available such as issues, discussions, releases, etc.
- `jobs` is a list of jobs to execute. Each job has a name, in this case `Hello-World`.
  - `runs-on` defines the runner ID. A common runner is using Ubuntu's latest version. This is actually a label for a runner provided and hosted by Github. Your organisation may run these runners within their own cloud infrastructure and provide custom labels. See this for [Github Hosted runner labels](https://docs.github.com/en/actions/writing-workflows/choosing-where-your-workflow-runs/choosing-the-runner-for-a-job#choosing-github-hosted-runners)
  - `steps` is a list of scripts to execute. Each element has a `run: <script>`. The script is a simple echo statement for this.

#### Commit and Push with VS Code

Now let's commit the change and push to Github. Click on the Source Control icon in the left hand navigation and click the `+` icon on the file while your workflow changes. 

![stage changes](img/vscode-stage.jpg)

Add a commit message and usin gthe drop down, commit and push the branch. 

![push](img/vscode-push.jpg)

On eht first push you will get the message asking to publish the branch. Click OK.

![publish](img/vscode-publish.jpg)

_Note:_ If you have Github extension installed, VS Code may prompt you to login using Github, Allow it and follow the prompts to authorize access. If you are using command line you will be prompted for Username and password.

#### Commit and Push via command line

You can commit and push via command line as follows:

```bash
git add .
git commit -m "hello world workflow added"
git push --set-upstream origin lesson
```

You need to add `--set-upstream origin lesson` the first time you run `git push` to specify the remote and branch name. Next time you can execute `git push`.

### Observe the Workflow

Soon after the push is successful, navigate to your browser and go to "Actions" tab on the repository page. You will see one action listed and being successful (Green tick).

![gh action](img/gh-helloworld-action.jpeg)

here you can see your workflows list (1) with the name taken from your YAML definition (it is not the filename). Secondly the table lists all the executions of this workflow. Each execution has a status (green tick or red cross). Clicking on the run will show you the detailed log of your build.

The first view shows the metadata and visual of the jobs and clicking on each job reveals the console log with any errors. 

![gh-jobs](img/gh-jobs.jpeg)

![gh-logs](img/gh-log.jpeg)

## 3. Python CI example

We've done the simple 'Hello, world' example but in reality we want the job to do something more useful. A common usecase is running unit tests on the python code in our repository. We will need a specific Python version and dependencies installed for our CI pipelines. The Ubuntu runner comes with Python pre-installed, however the version is not within our control and can change anytime without notice. To ensure a fixed version we can use the [`setup-python`](https://github.com/actions/setup-python) action that Github has provided. To use an action in our workflow we reference it using `uses` key and `with` object defining the input arguments.

```yaml
- uses: action/setup-python@v5
  with: 
    python-version: 3.11.x
```

To use this, create a new workflow file called `python-ci.yaml` in the `.github/workflows/` directory with the steps to install Python and then print the version of Python and pip. Commit and push this to Github.


```yaml
name: Python CI

on: [push,workflow_dispatch]

jobs:
  PythonVersionSetup:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Python
        # This is the version of the action for setting up Python, not the Python version.
        uses: actions/setup-python@v5
        with:
          # Semantic version range syntax or exact version of a Python version
          python-version: '3.11.x'
      - run: python --version
      - run: pip --version
```

![python ci](img/gh-pythonci-1.jpeg)

Next we will run pytest. However we don't yet have any python code to test! So let's add that. In the `data/` directory of this JupyterLab you will see `my-package.zip`. Download this and unzip it to your repository. The structure should look like

![my-package](img/vscode-package-structure.jpg)

We need to make 3 edits to make a unique package name update: Use lowercase and underscore only app_<username>

1. Edit `pyproject.toml` 
1. Edit the directory name to match. 
1. Edit `tests/test_add.py` and update `from app` -> `from app_<username>`

This will help us later to identify your package in registry. This is to avoid conflicts of two people trying to publish packages with same names and causing errors. When this happens one of you might receive an error saying the package already exists.

![pyproject edit](img/pyproject-edit.png)

Now we are ready to run tests in this package and build it. However we need to install Pytest first. So add two steps to install `pytest` and `pytest-cov` and run pytest with coverage report output:


```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install pytest pytest-cov
    pytest --doctest-modules --junitxml=junit/test-results.xml --cov=com --cov-report=xml --cov-report=html
```

![pytest fail](img/gh-pythonci-test-fail.jpeg)

Why did this fail? Firstly it didn't find any tests to run and secondly this mean there was no coverage report to generate. It failed with exit code 5 which means failure. Remember in linux, only exit code 0 is success.

This highlights an important behaviour of Github Actions runners. Your repository code is _not_ checked out automatically. The runner is only executing the scripts provided in our YAML or the action. We have not yet specific any git command to clone or checkout out repository. This behaviour is common with many CI/CD tools. 

Let's add a step to checkout the code as the first step:

```yaml
jobs:
  PythoVersionsSetup:
# ...
    steps:
      - name: Checkout
        uses: actions/checkout@v4
    # ... keep existing steps
```

![checkout test success](img/gh-checkout-test-success.png)

Now the code is checked out from git into the runner, and pytest is able to discover our 1 test and generate a test result report, Coverage report generation has failed with a warning due to no data.

As a checkpoint, let's clean up the workflow a little and s;
plit pytest command to a new step "Run Tests". Your Python CI workflow so far should look like:

```yaml
name: Python CI

on: [push, workflow_dispatch]

jobs:
  PythonVersionSetup:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Python
        # This is the version of the action for setting up Python, not the Python version.
        uses: actions/setup-python@v5
        with:
          # Semantic version range syntax or exact version of a Python version
          python-version: '3.11.x'
      - run: python --version
      - run: pip --version
      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest pytest-cov
      - name: Run Tests
        run: pytest --doctest-modules --junitxml=junit/test-results.xml --cov=com --cov-report=xml --cov-report=html
```

## 4. Poetry Build

Now that we have been able to run some pytests in Github Actions, let's now try to use poetry to build the package as a source distribution and wheel.

Firstly, we need to install poetry so add this to the existing "Install dependencies" step. Then add a new step to run the build:

```yaml
- name: Build
  working-directory: my_package
  run: poetry build
```

You will notice a new key `working-directory`. This allows us to change the working directory to any sub-directory within our repository.

![poetry build](img/gh-poetry-build.jpeg)

You can now see a pattern of adding new steps and how they are executed in order. Any failures will stop executing subsequent steps.

## 5. Artifacts and Publishing to PyPi registry

The next step is to upload the build artifacts to a PyPI registry. We will not be using pypi.org but a private registry setup specifically for this workshop. Unfortunately, Github Packages does _not_ support Python packages, so we have setup one in Gitlab. The registry URL you will use is `https://gitlab.com/api/v4/projects/61532089/packages/pypi` with the following credentials:

```yaml
username: cspark_user
password: glpat-zfBBtpCdQvrSJyW6vaBX
```

Back to Github Actions, let's add a new job to publish the packages. For publishing to PyPi, a prebuilt action [`pypa/gh-action-pypi-publish@release/v1`](https://github.com/pypa/gh-action-pypi-publish) is available. We will instead use `poetry` to publish it to our private registry.

```yaml
jobs:
  PublishPackage:
    run-on: ubuntu-latest
    steps:
    # copy the setup steps except Run Tests from previous job
      - name: Publish
        working-directory: my_package
        run: |
          poetry config repositories.cspark https://gitlab.com/api/v4/projects/61532089/packages/pypi
          poetry config http-basic.cspark cspark_user <password>
          poetry publish -r cspark 

```

**Before you commit**, adding the password as plain text to any git repository is a bad security practice. If you happen to commit any secrets to a git repository, please remove it and at your workplace, inform your manager and security teams. 

Github Actions provives a secure storage for secrets that can be referenced as environment variables. So let's go back to your repository in Github and click on Settings tab in the top navigation. Click on "Secrets and variables" on the left, then "Actions".

![gh settings](img/gh-settings.jpeg)

Click on "New repository secret" and add the password as `PYPI_TOKEN`:

![new secret](img/gh-new-repo-page.jpeg)

Once added, you will _not_ be able to view the secret value. When you click edit (pencil icon) the value field will be empty.

Go back to your workflow code to reference this password as `${{ secrets.PYPI_TOKEN }}`. This is known as accessing contexts. `secrets` is a special context for secrets defined in the repository or orgination. you can read more about [different contexts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs). The new job should look like this in full:

```yaml
  PublishPackage:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11.x'
      - run: python --version
      - run: pip --version
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install poetry
      - name: Publish
        working-directory: my_package
        run: |
          poetry config repositories.cspark https://gitlab.com/api/v4/projects/61532089/packages/pypi
          poetry config http-basic.cspark cspark_user ${{ secrets.PYPI_TOKEN }}
          poetry publish -r cspark
```

![publish fail](img/gh-publish-fail.jpeg)

The new job has failed. Remember that each job runs on a different runner, meaning jobs don't have context or access to files from other jobs. Secondly, both jobs have executed in parallel so there is no natural dependency between them.

Now modify the second job to add the key `needs` as follows before the steps:

```yaml
  PublishPackage:
    runs-on: ubuntu-latest
    needs: PythonVersionSetup
    steps:
    #....
```

Commit and push this change to see how the dependency is formed. Next we need to pass the files from first job to the next. For this we leverage Artifacts. These are files that are stored within the job and Actions context. The first job should upload artifacts and second job will download them. 

### Uploading Artifacts

An action available to upload artifacts is [`actions/upload-artifact@v4`](https://github.com/actions/upload-artifact). Only one input `path` is required while rest are optional. Using `name` input is recommended to allow to differentiate between multiple artifacts. Let's add a step to the `PythonVersionSetup` job:

```yaml
- name: Upload Build Artifacts
  uses: actions/upload-artifact@v4
  with:
    name: builds
    path: my_package/dist/*
```

Commit and publish the changes to see that the two jobs have dependencies and artifacts are uploaded. The PublishPackage job will still fail as we are yet to add the download step to it.

![Artifacts upload](img/gh-artifacts-upload.jpeg)

You can download the builds artifact which is a zip file containing the two packages (tar and whl files).

![builds](img/build-artifact.jpg)

### Downloading Artifacts

Now this artifact needs to be downloaded to the second job so that poetry can publish them to our private PyPi registry. Similar to `upload-artifacts` action, a [`actions/download-artifacts@v4`](https://github.com/actions/download-artifact) is also available. This action can take no input arguments and download all the artifacts. The `name` input can be used to donwload specific artifacts.

Add this step before the publish step in the `PublishPackage` job:

```yaml
- name: Download Build Artifacts
  uses: actions/download-artifact@v4
  with:
    name: builds
    path: my_package/dist/
```

We need the path to be within `my_package` as poetry requires the `pyproject.toml` file to be available to run any commands.

Add this command to "Install dependencies" step: `mkdir -p my_package/dist`. The directory needs to exist before artifacts can be downloaded. 

This should result in successful upload. if you get an error with `HTTP Error 400: Bad Request | b'{"message":"400 Bad request - Validation failed: File name has already been taken"}'` ensure you have renamed your package correctly (update pyproject.toml and directory names).

![publish success](img/gh-publish-success.jpeg)

## 5. Verify

To verify that the package has been published run the following pip command with your package's name instead of `app`.

```bash
pip download app --index-url https://cspark_user:glpat-zfBBtpCdQvrSJyW6vaBX@gitlab.com/api/v4/projects/61532089/packages/pypi/simple --trusted-host gitlab.com
```

This will download the wheel `.whl` file to your current directory. 

Expected Output:

```bash
Looking in indexes: https://cspark_user:****@gitlab.com/api/v4/projects/61532089/packages/pypi/simple
Collecting app
  Downloading https://gitlab.com/api/v4/projects/61532089/packages/pypi/files/0285b1ba54da84f4d57901fa59babdfff18cb556b5dd3d1b8749419f140a1498/app-0.1.0-py3-none-any.whl (1.2 kB)
Saved ./app-0.1.0-py3-none-any.whl
Successfully downloaded app

[notice] A new release of pip is available: 23.2.1 -> 24.2
[notice] To update, run: pip install --upgrade pip
```

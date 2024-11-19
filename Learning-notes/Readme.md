#### Introduction to GitHub Actions
---
GitHub Actions is a CI/CD pipeline directly integrated with your GitHub repository.

GitHub Actions allows you to automate:

Running test suites
Building images
Compiling static sites
Deploying code to servers
and more…

- GitHub Actions has templates you can use to get started.
- GitHub Actions files are defined as YAML files located in the **.github/workflow** folder in your repo.
- You can have multiple workflows in the repo triggered by different events.

Within a GitHub repo, you’ll have a tab for **Actions**.

#### Workflows
---
A workflow is a **configurable automated process** that will run one or more jobs.
Workflows are defined by a YAML file checked into your repository 

Workflows in your repo are defined in the following directory: **.github/workflows**

A repo can contain multiple workflows

Workflow triggers are events that cause a workflow to run:

- Events that occur in your workflow's repository
- Events that occur outside of GitHub and trigger a **repository_dispatch** event on GitHub
- Scheduled times
- Manual

The **name property** lets you easily identify Workflows.
```sh
name: macOS Workflow Example
# ... rest of workflow template
```

#### Workflow Components
---
The following GitHub Actions Workflows components:

- **Actions:** Reusable tasks that perform specific jobs within a workflow.
- **Workflows:** Automated processes defined in your repository that coordinate one or more jobs, triggered by events or on a schedule.
- **Jobs:** Groups of steps that execute on the same runner, typically running in parallel unless configured otherwise.
- **Steps:** Individual tasks within a job that run commands or actions sequentially.
- **Runs:** Instances of workflow execution triggered by events, representing the complete run-through of a workflow.
- **Runners:** Servers that host the environment where the jobs are executed, available as GitHub-hosted or self-hosted options.
- **Marketplace:** A platform to find and share reusable actions, enhancing workflow capabilities with community-developed tools.
- **Events:** Triggers that start workflows, such as `push`, `pull_request`, or `schedule`.
- **Secrets:** Encrypted values used to store sensitive data, like API keys, during workflow execution.
- **Artifacts:** Files or data created by workflows that can be saved and shared across jobs.
- **Cache:** Stores dependencies or files to reduce setup time in future workflow runs.
- **Matrix:** Allows a job to run with different configurations (e.g., OS, versions) simultaneously.
- **Environment Variables:** Data accessible throughout workflows, jobs, or steps.
- **Environments:** Define deployment stages (e.g., staging, production) and apply protection rules.
- **Contexts:** Dynamic information (e.g., branch name, job status) used within workflows.
- **Outputs:** Data passed from one job or step to another.
- **Permissions:** Control access to the repository and API for workflow actions.
- **Concurrency:** Manage multiple workflow runs, canceling or preventing concurrent executions.

#### Triggering Schedule Event
---
Schedule can use a cron expression to trigger a workflow at a specific time or day.

(on: This keyword defines the event that triggers a workflow to run. It specifies when the workflow should be executed, such as on push, pull_request, or a scheduled time.)

- This event will only trigger a workflow run if the workflow file is on the default branch.
- Scheduled workflows will only run on the default branch.
- In a public repository, scheduled workflows are automatically disabled when no repository activity has occurred in 60 days. For information on re-enabling a disabled workflow, see "Disabling and enabling a workflow."
- When the last user to commit to the cron schedule of a workflow is removed from the organization, the scheduled workflow will be disabled. If a user with write permissions to the repository makes a commit that changes the cron schedule, the scheduled workflow will be reactivated.

Example:

```sh
on:
  schedule:
    - cron: '30 5 * * 1,3'
    - cron: '30 5 * * 2,4'

jobs:
  test_schedule:
    runs-on: ubuntu-latest
    steps:
      - name: Not on Monday or Wednesday
        if: github.event.schedule != '30 5 * * 1,3'
        run: echo "Skip this step on Monday and Wednesday"
      - name: Every time
        run: echo "This step will always run"
```

You can use https://crontab.guru/ to translate time into a cron expression or Chatgpt.

#### Triggering Single or Multiple Events
---

- Single event. eg push
```sh
name: CI on Push

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run a one-line script
        run: echo "Hello, world!"
```

- Multiple events eg. push,pull request, release
```sh
name: CI on Multiple Events
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  release:
    types: [published, created]

jobs:
  build-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
```

If you specify multiple events, only one of those events needs to occur to trigger your workflow.
If multiple triggering events for your workflow occur at the same time, multiple workflow runs will be triggered.

#### Manual Events
---
You can trigger a workflow manually via the **GitHub UI, GitHub CLI, or GitHub REST API**

Using the gh workflow run command:

```sh
gh workflow run greet.yml \
-f name=mona \
-f greeting=hello \
-f data=@myfile.txt
```

You can specify the name, ID, or file name of the workflow you want to run

To run a workflow manually, the workflow must be configured to run on the **workflow_dispatch** event.

```sh
on:
  workflow_dispatch
    inputs:
      name:
        description: 'Name of the person to greet'
        required: true
        type: string
      greeting:
        description: 'Type of greeting'
        require: true
        type: string
      data:
        description: 'Base64 encoded content of a file'
        required: false
        type: string
```

You can define up to 10 inputs for a workflow_dispatch event.

Running the Workflow Manually in the **GitHub UI:**

After adding the workflow_dispatch event trigger to your workflow file, you can manually trigger the workflow from the GitHub UI by following these steps:

- Navigate to the Actions tab of your repository.
- On the left-hand side, select the specific workflow you want to run. If you don't see it, make sure to choose it from the list instead of leaving the default selection of "All workflows."
- You will see a notice indicating: "This workflow has a workflow_dispatch event trigger."
A Run Workflow button will appear. Clicking it will present a dropdown menu, allowing you to select the branch on which to run the workflow.
- If the workflow defines input parameters (e.g., name, greeting), you will be prompted to provide those values before running the workflow.

This method allows you to manually run your workflow directly from the UI without needing to use the GitHub CLI or REST API.

#### Webhook Events
---
Many of the listed GitHub Workflow Triggers are triggered by a webhook

What is a Webhook? A webhook is a public-facing URL that can be sent an HTTP request (often requiring authorization) to trigger events from external sources.

Most of these webhooks will be triggered within GitHub when users are interacting with GitHub which will in turn will trigger API actions. Users generally don’t have to directly call the API to trigger the workflow.

External Webhook Events
Using **repository_dispatch** with webhook type you can trigger the Workflow via an external HTTP endpoint.

will only trigger a workflow run if the workflow file is on the default branch

```sh
name: Workflow on Repository Dispatch

on:
  repository_dispatch:
    types:
    - webhook

jobs:
  respond-to-dispatch:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
    - name: Run a script
      run: echo "Event of type ${GITHUB_EVENT_NAME}"
```

When you make the request to the webhook, you must:

- Send a POST request to the rep’s dispatches endpoint
- Set the Accept type for application/vnd.github+json
- Provide Authorization for your Personal Access Token
- Pass the event type ”webhook”

```sh
curl -X POST \
-H "Accept: application/vnd.github+json" \
-H "Authorization: token {personal token with repo access}" \
-d '{"event_type": "webhook", "client_payload": {"key": "value"}}' \
https://api.github.com/repos/{owner}/{repo}/dispatches
```

#### Conditional Keywords for Steps
---
jobs..if conditional can be used to prevent a job from running unless a condition is met.

```sh
name: example-workflow
on:[push]
jobs:
  production-deploy:
    if: github.repository == 'octo-org/octo-repo-prod'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
```

#### Runners
----
The Runner determines the underlying compute and OS that the the workflow will execute on.

The runner can be:

- **GitHub-hosted** — GitHub providers predefined runtime environments
  - Standard size
    - Linux: ubuntu-latest, ubuntu-22.04, ubuntu-20.04
    - Windows: windows-latest, windows-2022, windows-2019
    - MacOS: macos-latest, macos-14, macos-13, macos-12, macos-11
  - Larger Size
    - Only available for organizations and enterprises using the GitHub
    - Team or GitHub Enterprise Cloud plans
    - More RAM, CPU, and disk space
    - Static IP addresses
    - The ability to group runners
    - Autoscaling to support concurrent workflows

- **Self-hosted** — external compute connected to GitHub using the GitHub Actions self-hosted runner application
create custom hardware configurations that meet your needs

Use the **runs-on** to specify the runner

```sh
# specify a specific GitHub-self hosted
runs-on: ubuntu-latest
runs-on: windows-latest
runs-on: macos-latest

# specify multiple possible runners
runs-on: [macos-14, macos-13, macos-12]

# specify Self-Hosted runner
runs-on: self-hosted
```

If you specify an array of strings or variables, your workflow will execute on any runner that matches all of the specified runs-on values.

#### Self-Hosted Runner
---
Self-hosted runners can be:

- Physical
- Virtual
- In a container
- On-premises
- In a cloud

You can add self-hosted runners at various levels in the management hierarchy:

- Repository-level runners are dedicated to a single repository.
- Organization-level runners can process jobs for multiple repositories in an organization.
- Enterprise-level runners can be assigned to multiple organizations in an enterprise account.

To set up self-hosted, you need to add a runner and install the **GitHub Actions Runner** to connect the external compute to the self-hosted runner.
After installing the GitHub Actions Runner, ensure **port 443** is open in your outbound rules to allow secure communication between the runner and GitHub.

#### Workflow Commands Summary
---
Workflow commands in GitHub Actions let you interact with the runner machine during a workflow, performing tasks like setting environment variables, modifying the system PATH, sending debug messages, or sharing data between steps/jobs.

- Set Environment Variables
Add variables for subsequent steps using $GITHUB_ENV:

```yaml
steps:
  - name: Set environment variable
    run: echo "ACTION_ENV=production" >> $GITHUB_ENV
```

- Add to System PATH
Append a directory to the PATH variable using $GITHUB_PATH:

```yaml
steps:
  - name: Add directory to PATH
    run: echo "/path/to/dir" >> $GITHUB_PATH
```

- Set Output Parameters
Share data between steps or jobs using $GITHUB_OUTPUT:

```yaml
steps:
  - name: Set output
    id: example_step
    run: echo "result=output_value" >> $GITHUB_OUTPUT
  - name: Use output
    run: echo "The output was ${{ steps.example_step.outputs.result }}"
```

- Create Debug Messages
Send debug logs visible only in debug mode:

```yaml
steps:
  - name: Debug message
    run: echo "::debug::This is a debug message"
```

Key Points:

**$GITHUB_ENV, $GITHUB_PATH, and $GITHUB_OUTPUT** are special files for managing variables, PATH, and outputs.
Use these commands to efficiently share data for the current workflow run or customize the runner environment during workflows.

#### Workflow Contexts
---
Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps.

Each context is an object that contains properties, which can be strings or other objects.

You can access contexts using the expression syntax. **${{ <context> }}**

```sh
steps:
  - name: Use Secret as Env Var
    run: echo "Secret Value: $MY_SECRET"
    env:
      MY_SECRET: ${{ secrets.MY_SECRET }}
```

- github — Information about the workflow run.
- env — Contains variables set in a workflow, job, or step.
- vars — Contains variables set at the repository, organization, or environment levels.
- job — Information about the currently running job.
- jobs — For reusable workflows only, contains outputs of jobs from the reusable workflow.
- steps — Information about the steps that have been run in the current job.
- Runner — information about the runner that is running the current job.
- secrets — Contains the names and values of secrets that are available to a workflow run.
- strategy — Information about the matrix execution strategy for the current job.
- matrix — Contains the matrix properties defined in the workflow that apply to the current job.
- needs — Contains the **outputs** of all jobs that are defined as a dependency of the current job.
- inputs — Contains the **inputs** of a reusable or manually triggered workflow.

#### Dependent Jobs
---
A workflow run is made up of one or more jobs, which run in parallel by default. To run jobs sequentially, you can define dependencies between them using:

- jobs..needs

```sh
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

- jobs..if

```sh 
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

#### Encrypted Secrets
---
**Encrypted Secrets** are variables that allow you to **pass sensitive information** to your GitHub Actions Workflow

Secrets are accessed via the **secrets context eg. ${{ secrets.MY_SECRET }}**

- **Organization-Level Secrets**
you can use access policies to control which repositories can use organization secrets to share secrets between multiple repositories
Updating organization secrets propagates changes to all shared repos.

- **Repository-level Secrets**
Secrets that are shared across all environments for a repo.

- **Environment-level Secrets**
You can enable required reviewers to control access to the secrets

Lower-level env vars override high-level env vars

**NOTE:** Secret names can only contain alphanumeric characters and underscores. No spaces allowed eg. Hello_world123
- Names must not start with GITHUB_ prefix
- Names must not start with numbers
- Names are case-insensitive
- Names must be unique at the level they are created at

**Encrypted Secrets — Accessing Secrets**
- Passing Secrets as Inputs
You can pass secrets as inputs by using the **secrets context**.

```yml
name: Example of Using Secret as Input

on: [push]

jobs:
  use-secret-as-input:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Custom Action using Secret
        uses: example/action@v1
        with:
          api-key: ${{ secrets.API_KEY }}
```

- Passing Secrets as **Env Vars**
You can also pass secrets to actions or scripts by setting them as environment variables.

```yml
name: Example Using Secret as Env Var

on: [push]

jobs:
  use-secret-as-env:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run a script using Secret
        run: |
          echo "Using API key: $API_KEY"
        env:
          API_KEY: ${{ secrets. API_KEY }}
```

**Encrypted Secrets — Settings Secrets**
- Settings Secrets at the Repository-level
```sh
# set a secret and prompt for secret value
gh secret set SECRET_NAME
# sets a secret and sets values from a text file
gh secret set SECRET_NAME < secret.txt
```

- Settings Secrets at the Environment-level
```sh
# set a secret and prompt for secret value
gh secret set --env ENV_NAME SECRET_NAME
# get list of secrets for an env
gh secret list --env ENV_NAME
```

- Settings Secrets at the Organization-level
```sh
# login with admin:org scope to manage org secrets
gh auth login --scopes "admin:org"
# set a secret only for private repos and prompt for secret value
gh secret set --org ORG_NAME SECRET_NAME
# set a secret for public, private, and internal repos
gh secret set --org ORG_NAME SECRET_NAME --visibility all
# set a secret for specific repos
gh secret set --org ORG_NAME SECRET_NAME --repos REPO-NAME-1, REPO-NAME-2
# list secrets for the org
gh secret list --org ORG_NAME
```

#### Configuration Variables
---
Configuration Variables are variables that allow you to **pass non-sensitive information** to your GitHub Actions Workflow
variables are accessed via the **variable context eg.${{ vars.VAR_NAME }}**

They can be set at various levels:
- Repository-level
- Environment-level
- Organization-level

**Key Features of Variables**
- Used to parameterize workflows and avoid hardcoding.
- Can be updated centrally, affecting all workflows relying on them.

Examples:

```bash
gh variable set MYVARIABLE                                          #repository level
gh variable set MYVARIABLE --env myenvironment                      #environment level like staging, production, devlopement etc
gh variable set MYVARIABLE --org myOrg --repos repo1,repo2          #organization level 
```

#### Default Env Vars
The **default environment variables that GitHub sets** are available at every step in a workflow.
example: CI, GITHUB_ACTION, GITHUB_ACTOR, GITHUB_ACTOR_ID, GITHUB_ENV, GITHUB_JOB, GITHUB_REF ....... etc
You can find the complete list of these default environment variables and their explanations here [GitHub env DOC](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables#default-environment-variables)


```yml
name: Example Workflow Using Default Env Variables

on: [push]

jobs:
  example_job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Print GitHub Environment Variables
        run: |
        echo "Repository Name: $GITHUB_REPOSITORY"
        echo "Workflow: $GITHUB_WORKFLOW"
        echo "Action: $GITHUB_ACTION"
        echo "Actor: $GITHUB_ACTOR"
```

#### Set Custom Env Vars
---
You can define Environment Variables inline within your GitHub Actions workflow.

```yml
name: Greeting on variable day

on:
  workflow_dispatch

env:
  DAY_OF_WEEK: Monday                                               #workflow level

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      Greeting: Hello                                               #job level        
    steps:
      - name: "Say Hello Mona it's Monday"
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
        env:
          First_Name: Mona                                          #step level
```

- **Workflow-level**
Set env vars of the entire workflow
- **Job-level**
Set env vars for the entire job
- **Step-level**
Set env vars for the step

We can also access env vars via the **env context**

#### Set Env Vars with Workflow Commands
---
In GitHub Actions, you can dynamically set env vars during the execution of your workflows using the **$GITHUB_ENV** special workflow command.

This is useful for passing values between steps, dynamically adjusting behavior based on runtime data, or configuring tools and scripts executed by your workflow.

```yml
name: Set Environment Variables Example

on: [push]

jobs:
  setup-and-use-env:
    runs-on: ubuntu-latest
    steps:
      - name: Set dynamic environment variable
        run: |
          # Using workflow command to set an environment variable
          echo "DYNAMIC_VAR=Hello from GitHub Actions" > $GITHUB_ENV

      - name: Use the environment variable
        run: |
          # Using the environment variable in a subsequent step
          echo "The value of DYNAMIC_VAR is: $DYNAMIC_VAR"
```
Anything placed into $GITHUB_ENV will be accessible anywhere in your workflow.
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
---
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

#### GITHUB_TOKEN Secret
---
At the start of each workflow job, GitHub automatically creates a unique GITHUBTOKEN secret to use in your workflow. You can use the GITHUBTOKEN to authenticate in the workflow job.

When you enable GitHub Actions, GitHub installs a GitHub App on your repository.

The **GITHUB_TOKEN** secret is a GitHub App installation access token.

```yml
name: Open new issue
on: workflow_dispatch

jobs:
  open-issue:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
    steps:
      - run: |
        gh issue --repo ${{ github.repository }} \
          create --title "Issue title" --body "Issue body"
        env:
          GH_TOKEN: $ {{ secrets.GITHUB_TOKEN }}
```

You can use the GITHUB_TOKEN to make authenticated API calls.

When is GITHUB_TOKEN Required?
- Push Changes to a Repository
  To commit and push workflow-generated changes back to the repository.

- Trigger Subsequent Workflows
  To trigger other workflows programmatically (e.g., dispatch events).

- Use GitHub REST or GraphQL APIs
  For operations like creating issues, managing releases, or triggering workflows.

- Access Private Repositories
  For interacting with private repositories, such as checking out code or fetching submodules.

- Running Actions That Modify Repository Settings
  To update branch protection rules, repository secrets, or collaborator permissions.

- Uploading or Downloading Artifacts
  To securely store or retrieve artifacts during workflow execution.

#### Add Script to Workflow
---
You can execute bash scripts within a GitHub Actions workflow

```yml
jobs:
  example-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./scripts
    steps:
      - name: Check out the repository to the runner
        uses: actions/checkout@v4
      - name: Run a script
        run: ./my-script.sh
      - name: Run another script
        run: ./my-other-script.sh
```

#### Publish GitHub Package using Workflow
---
You can use a workflow to build a GitHub Package
( GitHub Packages is a service offered by GitHub for hosting and managing packages, such as dependencies or container images, within your GitHub repository. It allows you to store and share various types of packages (e.g., npm, Maven, Docker, RubyGems) directly in GitHub. You can integrate GitHub Packages with workflows in GitHub Actions for automating your CI/CD pipelines. )

```yml
name: Create and publish a Docker image
on:
  push:
    branches: ['release']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Log in the Container registry
        uses: docker/login-action@65b78...
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57...
        with:
          images: $ {{ env.REGISTRY }} /${{ env.IMAGE_NAME }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d...
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

#### Publish Docker Hub Registry using Workflow
---
You can use a workflow to build and publish to Docker Hub Registry

```yml
name: Publish Docker image

on:
  release:
    types: [published]

jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses docker/login-action@f4e78...
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57...
        with:
          images: my-docker-hub-namespace/my-docker-hub-repository

      - name: Build and push Docker image
        uses: docker/build-push-action@3b5e8...
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

#### Publish GitHub Container Registry using Workflow
---
Pushing container to the GitHub Container Registry

```yml
name: Publish Docker Image

on: [push]

jobs:
  build-and-push
    runs-on: ubuntu-latest
    steps:
    - name: Check out the repo
      uses: actions/checkout@v2
    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.CR_PAT }} # or use GITHUB_TOKEN
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1
    - name: Build and push Docker image
      users: docker/build-push-action@v2
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ghcr.io/${{ github.repository_owner }}/my-image latest
    - name: Verify the image was pushed
      run: docker pull ghcr.io/${{ github.repository_owner }}/my-image latest
```

#### Publish Component as GitHub Release
---
Publishing Components as GitHub Release

```yml
name: Release Workflow
on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
steps:
  - name: Checkout code
    uses: actions/checkout@v2

  - name: Build project
    run: |
      echo "Build your project here"
      # Example: gcc -o output_binary source_code.c

  - name: Create Release
    id: create_release
    uses: actions/create-release@v1
    env:
      GITHUB_TOKEN: $ {{ secrets.GITHUB_TOKEN }}
    with:
      tag_name: ${{ github.ref }}
      release_name: Release ${{ github.ref }}
      draft: false
      prerelease: false

  - name: Upload Release Asset
    uses: actions/upload-release-asset@v1
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      upload_url: ${{ steps.create.release.outputs.upload_url }}
      asset_path: ./path/to/your/output_binary
      asset_name: output_binary_name
      asset_content_type: application/octet-stream
```

A **release** in GitHub is a versioned snapshot of your repository. It typically represents a stable version of your code and includes a Git tag, release notes, and optional files (assets) such as binaries or build artifacts. Releases make it easy for others to download, use, or deploy your project. Using GitHub Actions, you can automate the creation and publishing of GitHub releases with predefined actions like actions/create-release for generating the release and actions/upload-release-asset for attaching assets.

#### Deploy Release to Cloud Provider
---
You can deploy releases to specific CSP:

- Amazon Elastic Container Service (ECS)
- Google Kubernetes Engine
- Azure App Services, Azure Kubernetes Service (EKS), Azure Static Web Apps

#### Service Containers
---
A container service is a temporary containerized environment that runs alongside your workflow jobs. It is typically used to provide services (like a database or cache) that your application or tests require to function properly.

Key Points:
- Lifecycle: The service container starts when the job begins and stops when the job ends.
- Isolation: It runs in isolation from the main job environment, providing a clean and reproducible setup for testing or development.
- Configuration: Define the service in your workflow under the services key, specifying the image, environment variables, ports, and health checks.
- Access: The service is accessible via localhost or specified ports.

Why and When to Use It:
- For Testing: To simulate dependencies like a database (MySQL/PostgreSQL), caching service (Redis), or message queue (RabbitMQ) in a controlled environment.
- For Integration: Test how your application interacts with external systems without needing to connect to production services.
- For Reproducibility: Ensure consistent testing by spinning up a fresh service instance every time a workflow runs.

By using service containers, you can test, debug, and validate your workflows in isolated and predictable environments.

```yml
name: PostgreSQL Container Service
on: push

jobs:
  container-job:
    runs-on: ubuntu-latest
    container: node:14
    services:
      postgres:
        image: postgres
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - name: Check out repository code
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          cd ./github-actions/pg
          npm install

      - name: Connect to PostgreSQL
        run: node ./github-actions/pg/client.js
        env:
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
```

note: You can use the **credentials** keyword if your container image requires authentication to pull from a private registry.
```yml
services:
  my-service:
    image: private-registry/my-image:latest
    credentials:
      username: ${{ secrets.DOCKERHUB_USERNAME }}
      password: ${{ secrets.DOCKERHUB_PASSWORD }}
```

#### Routing Workflow to Runner
---
Routing workflows to runners involves specifying where a GitHub Actions workflow should execute. Runners are servers that run your workflow jobs, and they can be hosted by GitHub (e.g., ubuntu-latest, windows-latest) or self-hosted (custom hardware or VMs set up by you).

A self hosted runner automatically receives certain lables when it is added to github actions

**Default Labels:**
- self-hosted - Indicates it is a self-hosted runner.
- OS-specific (e.g., linux, windows, or macOS).
- Architecture (e.g., x64, arm64).

You can also add **custom labels** during or after setup to make routing more specific (e.g., high-memory, gpu). For instance:

```yaml
runs-on: [self-hosted, linux, x64, gpu]

``` 

A self hosted runner must have all four labels to be eligible to process the job.

**Runner groups** are used to collect sets of runners and create a security boundary around them. Only Entirprise accounts, organizations owned by enterprise accounts, and organizations using GitHub Team can create and manage additional runner groups.

```yml
runs-on: [self-hosted, linux, group_name]
```

#### CodeQL Step
---
CodeQL into a Github Actions workflow can automate the process of code analysis, allowing you to find and fix security issues before they are exploited. 

you can set up CodeQL analysis in GitHub Actions using predefined actions like github/codeql-action/init@v1 and github/codeql-action/analyze@v1. These actions help integrate CodeQL into your GitHub workflows, enabling security analysis for your codebase.

```yml
jobs:
  analyze:
    name: Analyze code with CodeQL
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up CodeQL
        uses: github/codeql-action/init@v1
        with:
          languages: 'javascript'  # Add other languages if needed

      - name: Perform CodeQL analysis
        uses: github/codeql-action/analyze@v1
```

#### Caching Package and Dependency Files
---
To make your workflow faster and more efficient, you can create and use caches for dependencies and other commonly reused files.

The following package managers and and their Action:
- npm, yarn, pnpm: setup-node
- pip, pipenv, Poetry: setuppython
- Gradle, Maven: setup-java
- RubyGems: setup-ruby
- Go go.sum: setup-go
 
```yml
- name: Set up Ruby
  uses: ruby/setup-ruby@v1
  with:
    bundler-cache: true

- name: Cache Ruby dependencies
  uses: actions/cache@v2
  with:
    path: ~/.bundle
    key: ${{ runner.os }}-ruby-${{ hashFiles('**/*.gemspec') }}
    restore-keys: |
      ${{ runner.os }}-ruby-
```

- **actions/setup-\*** installs and configures the necessary language or environment for the project.
- **actions/cache** is used for caching the dependencies so that future workflow runs can restore these dependencies without needing to download or rebuild them.


#### Remove Workflow Artifact from GitHub
---
Artifact typically refers to build outputs or files generated during the workflow that you may want to store or use later.

To delete the workflow artifact you need to delete them via UI , once you delete it cannot be restored.

by default github stores build and artifacts for 90 days, and its retention period can be customized.

#### Workflow Status Badge
---
Workflow badges in GitHub Actions are visual indicators that display the current status of a workflow, such as whether it passed or failed. These badges are typically displayed in the README file in repository.

To add a workflow badge:

- Go to the Actions tab of your GitHub repository.
- Select the desired workflow.
- Click on the "badge" icon to copy the markdown snippet, or manually create a badge using the format:

```md
![Build Status](https://img.shields.io/github/workflow/status/your-username/your-repo/your-workflow-name)
```

This will display the status of the workflow in your repository's README or any other markdown file.


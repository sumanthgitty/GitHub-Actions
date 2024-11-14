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



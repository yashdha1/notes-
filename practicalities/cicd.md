
# pre-commit hooks 

before even committing the code : runs the ruff ,isort, black and all the testcases in each services to make sure the code follows a standard and 100% all the testcases. 

setup is 
uv add pre-commit 
make a file with example configuration as :  

".pre-commit-config.yaml"

```yaml 
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
    -   id: black
-   repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.10
    hooks:
    -   id: ruff
-   repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
    -   id: isort
-   repo: local 
  # here will the configurations for all the test cases running : 
```

run the pre-commit files with either :--> **uv run pre-commit run --all-files**

or the hook run automatically on the 
git commit :-> making sure that the code you have written is correect and does not break anything. 

--- 

# Git branching strategies : 
- `main` → production-ready code
- `develop` → integration branch
- `feature/*` → new features
- `release/*` → preparing releases
- `hotfix/*` → urgent production fixes

# CI CD and CD
- **Continuous Integration (CI):** Focuses on automatically building and testing code changes. When you push code or open a pull request, GitHub Actions runs your test suite. This catches bugs early and ensures the shared repository remains stable. Runner picks up the code and runs a array of things on this code. 
- **Continuous Delivery (CD):** Automatically prepares the tested code for a manual release to production-ready environments.
- **Continuous Deployment (CD):** Takes automation a step further by automatically deploying every change that passes tests directly to your customers (e.g., to AWS, Azure, or GitHub Pages) without human intervention.


Continue this video only via the WSL in the windows :-> 

https://www.youtube.com/watch?v=Xwpi0ITkL3U 

Workflow and Github actions templates : https://docs.github.com/en/actions/get-started/understand-github-actions




You can do array of things on certain events. Pull req or push request. called as triggers, on the trigger we can do certain shit and proceed accrodingly. run testcases, autocheck the code quality. 


Allows us to automate taks and workflows to proceed and have clearner integrations. GitHub Actions is an automation tool that allows developers to create CI/CD (Continuous Integration/Continuous Deployment) pipelines. It helps automate various development tasks like testing, building, and deploying applications. 


Githuv workflow is bunch of automation files, 

A GitHub Workflow is a set of automated tasks defined in a *YAML file*. t runs when **triggered** by specific events, such as pushing code to a repository, creating a pull request, or setting up a schedule. 

**Workflows help in automating repetitive tasks like testing, code analysis, and deployment.** 


--- 

* ***Github Runner:-** 

A GitHub Runner is a virtual machine that executes jobs specified in a GitHub Actions workflow. GitHub provides both hosted runners (free-tier available) and self-hosted runners (custom machines for private execution).

---
### **Workflows** 

A **workflow** is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.  


Workflows are defined in the `.github/workflows` directory in a repository. A repository can have multiple workflows, each of which can perform a different set of tasks such as: 

- Building and testing pull requests
- Deploying your application every time a release is created
- Adding a label whenever a new issue is opened
---

### **Events** 

An **event** is a specific activity in a repository that triggers a **workflow** run. Now these can be push based or pull based. In modern terms these are usually pull based. but whenever commit we run a pre-commit hooks to run the. the linters, static code analysers and tests. 
eg. black, ruff, isort and testcases. (No integrations run at this stage). 

there are array of events on which you can trigger a workflow: https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#about-events-that-trigger-workflows

example: events on which workflows can be triggered. 

|Event|Triggers When...|
|---|---|
|`push`|Commits are pushed to any branch|
|`create`|A branch or tag is created|
|`delete`|A branch or tag is deleted|
|`release`|A release is published, edited, deleted|
|`fork`|Someone forks the repo|
**Pull Request Events** 

| Event                         | Triggers When...                             |
| ----------------------------- | -------------------------------------------- |
| `pull_request`                | PR opened, closed, merged, labeled, reviewed |
| `pull_request_review`         | A review is submitted, dismissed             |
| `pull_request_review_comment` | A comment is added to a PR diff              |

You can protect certain branches when pulling form the dev or any kind of malicious or incorrect commit. eg. 

whenever merging in the main or dev we can have the mergeing code run through array of test suites or workflow to verify the code. 


workflow has bunch of **jobs** that they need to run. 

--- 


### Jobs

A **job** is a set of **steps** in a workflow that is executed on the same **runner**. Each step is either a shell script that will be executed, or an **action** that will be run. Steps are executed in order and are dependent on each other. Since each step is executed on the same runner, you can share data from one step to another. For example, you can have a step that builds your application followed by a step that tests the application that was built.

You can configure a job's dependencies with other jobs; by default, jobs have no dependencies and run in parallel. When a job takes a dependency on another job, it waits for the dependent job to complete before running. so like it can health check or check the completion of jobs to proceed further. 


### Action 

Set of jobs are c/as actions. An **action** is a pre-defined, reusable set of jobs or code that performs specific tasks within a **workflow**, reducing the amount of repetitive code you write in your workflow files. Actions can perform tasks such as:

- Pulling your Git repository from GitHub
- Setting up the correct toolchain for your build environment
- Setting up authentication to your cloud provider

### Runner 
runs all the actions and the jobs. It itself is a vm that does this. This is provided freely by the github themself and can be configured at a cost. these can be free, you can make yours or can be taken from the github. 

![[Pasted image 20260413121208.png]]

from the runner we do array of actions. 



useing the localstack for learning purposes for now: 



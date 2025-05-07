# Chapter 14. Migrating to GitHub Actions

As you’ve seen throughout the book, GitHub Actions provides a powerful platform for integrating automation with GitHub. The workflows and actions can be used to do any of the typical types of tasks, such as CI/CD, that you may currently be doing with another tool or platform. And, when you are using a different platform, migrating to GitHub Actions can seem like a daunting proposition.

In this chapter, I’ll cover the basics you need to know to work on migrating from a set of selected CI/CD platforms to GitHub Actions. I’ll also show you how to use a new tool—the *GitHub Actions Importer*—that can help get you part of the way there with automation.

GitHub Actions can be implemented to replace any current automation framework, though some may need more customization than others. As a general rule, if you are using one of these six options, the migration is more straightforward. The importer tool is also specifically designed to work with pipelines in these platforms:

* Azure DevOps
* Bamboo (in beta, see note)
* CircleCI
* GitLab
* Jenkins
* Travis CI

# Support for Bamboo

Support for Atlassian Bamboo was added after this chapter was written and is currently in beta as of the time of this writing. For these reasons, details of migrating from it are not fully covered here. For more information on it, see [the documentation](https://oreil.ly/zMRAL).

There isn’t time and space in this chapter to cover all of the details. But I’ll give you enough to get going; refer to locations in the GitHub Actions documentation for more detailed information for each platform. For each of these types (except Bamboo), I’ll note the similarities, share a table that highlights the differences, and then go through a brief example.

# Legal Notice

Portions of the examples in this chapter have been adapted from the [GitHub Actions Importer page](https://oreil.ly/Xj-hD) under the [MIT license](https://oreil.ly/7K2Yk).

# Prep

Before any migration, there are important steps to think about and plan for. These include reviewing your source code, your automation, your infrastructure, and your users.

## Source Code

Actions and workflows are associated with GitHub repositories. So your source code should live in, or be migrated to, GitHub repositories before moving to Actions for automation. The mechanics of this are simple if you already use/know Git. And GitHub will even provide the explicit set of instructions to accomplish this if you create an empty repository ([Figure 14-1](#instructions-in-a-new)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1401.png)<br>
**Figure 14-1. Instructions in a new GitHub repo**

There are several items to consider and address before just moving your code into GitHub as is, though. Consider the following questions, and make sure you are comfortable with the answers before proceeding:

* Do you want to move all of your source code or only certain projects?
* Do you want to move all of the history or only the most recent content?
* Do you need to move all of the branches or only a subset?
* Do you need to delete any content or do any other cleanup?

If you happen to be coming at this from repositories that are not in Git, you will also need to think about what kind of structure makes the most sense in Git and not just convert them (necessarily) as is. All of the considerations around moving to Git are beyond the scope of this chapter, but in general, remember that Git works best for multiple, smaller repositories versus larger monolithic ones.

In short, this is a good time to go through and clean up/simplify your set of source projects. The same applies for your automation.

## Automation

This part of the conversion is the main focus of this chapter. As with the other areas, there is prep work that is essential to ensuring as smooth a migration as possible. Here are some questions to think about in this area:

* Do you want to move automation for all projects or only a subset?
* If the reason for a part of your existing automation is not obvious, do you have resources that can explain what the purpose of the automation is? If not, can you at least identify whether it is still essential or not?
* Can you identify from logging or other sources which automation is no longer used or run so infrequently that it may not make sense to move it?
* Can you identify any calls to custom scripting or kludges that have been added? These will likely be more difficult to convert *if* they need to be carried forward.
* Is your current automation using the latest releases of supporting plug-ins, orbs, or other modules? The same evaluation applies for any custom supporting pieces written *in-house*. Do you know what those supporting pieces are so you can find an equivalent action to replace them?
* Are there any places where your current automation is broken or requires manual intervention? Can those be fixed so the same challenge doesn’t carry over past the migration?

The more you can do prior to the actual migration to standardize, simplify, and understand your automation, the better off you’ll be when you undertake it. The same holds true for your infrastructure, especially since there can be an added element for that—cost.

## Infrastructure

As you convert your automation, you will need to specify where the jobs in your new workflows will run—on a GitHub-hosted runner or on a self-hosted runner. So you need to think about that before you have to migrate.

Runners are covered in detail in this book in [Chapter 5](ch05.html#ch05). As noted there, there can be a cost factor if you’re not using public repositories and/or self-hosted runners. Whether you’re using GitHub’s provided runners or your own self-hosted ones, this is a good time to consider the following items:

* Are there any custom setups/configurations that will need to be migrated?
* Are there any custom applications, specific versions of applications, or specific versions of operating systems required to be set up on the machines?
* If you are intending to use GitHub runners, can each job that will be migrated tolerate being run on a VM that will be unavailable before and after the job is run?
* If you are intending to use GitHub runners and you currently use Mac or Win­dows systems, is that still required, or can you switch to Linux environments to avoid additional costs?

Last, but certainly not least, you need to ensure that your users will have access to all of the migrated content.

## Users

Planning for the impact of the migration on your users (those who need any kind of access to the source code, automation, or infrastructure being migrated) needs to be one of the main things you think about while going through the other parts of the process. Points to consider in this planning space include the following:

* What are the appropriate permissions/accesses for team members to have to this code? Who are the administrators? Who are the contributors? Is this also a time to clean those permissions and assignments up?
* Are the users who will be affected advised of the change and the timing of it? Do they all have GitHub accounts?
* Do all users have appropriate training (or pointers to training on GitHub and GitHub Actions)?

While this doesn’t have to be the first area you complete, it does need to be the last area you ensure is complete before migrating. When you pull the switch to change to using GitHub Actions, you want to ensure that it’s as smooth and painless as possible for your users.

So how do you ensure that things go as smoothly as possible during the actual migration? No process is perfect, but here are a few suggestions:

* Remove any unneeded/outdated source code, automation, infrastructure, users, or user accesses to reduce the amount of work needed.
* Standardize as much as possible existing source code, automation, infrastructure, users, and user accesses to make migration more straightforward and repeatable.
* Ensure you are compliant with any security mandates and that users are aware of what’s required for compliance and why.
* Allow sufficient time to do the migration, expecting that questions and problems will come up along the way.
* Track the process formally so it’s clear what stage of migration each repository is in.
* Require appropriate training for all team members that will be working with the migrated content.
* Set up a test conversion of all of the pieces associated with one repository as soon as possible and have those that will need to work with the migrated content access it. They should also run through/demonstrate proficiency with simple tasks such as making a source change, doing a pull request, and looking at the automation run in GitHub Actions.

In the next sections of this chapter, we’ll look in more detail at conversions from some of the popular platforms to GitHub Actions.

# Azure Pipelines

Azure Pipelines is one part of a suite of developer services that can be used to streamline software development tasks such as planning, collaboration, deployment, etc. Using it is similar to using GitHub Actions in the following ways:

* Configuration files for workflows are authored with YAML.
* Configuration files for workflows are stored with the repository code.
* Workflows include jobs.
* Jobs include steps that run sequentially.
* Jobs run on separate VMs or in separate containers.
* Jobs run in parallel by default but can be sequenced.
* Steps can be shared with a community and reused.

Here is an example Azure pipeline listing:

```yml
# Simple pipeline to build a Node.js project with React
# Add steps to customize
# https://docs.microsoft.com/azure/devops/pipelines/languages
# /javascript

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '16.x'
  displayName: 'Install Node.js'

- script: |
    npm install
    npm run build
  displayName: 'npm install & build'
```

There are also key differences between Azure Pipelines and GitHub Actions, summarized in [Table 14-1](#key-differences-betwe).

Table 14-1. Key differences between Azure Pipelines and GitHub Actions workflows

| Category | Azure Pipelines | GitHub Actions workflows |
| --- | --- | --- |
| Editor | GUI editor | YAML spec |
| Job spec | Not needed if only one job | Required for all jobs |
| Combined workflows | Can use stages to define multiple workflows in same file | Requires separate files for each workflow |
| On-prem runs | Selected via build agents with capabilities | Selected via self-hosted runners with labels |
| Script step keywords | *script, bash, powershell, pwsh* | *run* (with *shell* if needed for particular shell) |
| Script error handling | Can configure to error if any output sent to *stderr;* requires explicit config to exit immediately on error | Enacts *fail fast* approach, stopping workflow immediately on error |
| Default Windows shell | Command shell (*cmd.exe*) | PowerShell |
| Trigger keyword | *trigger* | *on* |
| OS definition keyword | *vmImage* | *runs-on* |
| Conditional keyword | *condition* | *if* |
| Conditional execution syntax | expression (i.e., *eq*) | infix/operator (i.e., *==*) |
| Sequential execution keyword | *dependsOn* | *needs* |
| Reusable components | tasks | actions |
| Reusable component keyword | *task* | *users* |
| Name keyword | *displayName* | *name* |

The following listing shows an example conversion of the previous Azure Pipelines to a comparable GitHub Actions workflow:

```yml
name: demo-pipeline
on:
  push:
    branches:
    - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: checkout
      uses: actions/checkout@v4.2.2
    - name: Install Node.js
      uses: actions/setup-node@v4.4.0
      with:
        node-version: 16.x
    - name: npm install & build
      run: |-
        npm install
        npm run build
```

More details on converting from Azure Pipelines to GitHub Actions can be found [in the documentation](https://oreil.ly/JjAPv).

# CircleCI

CircleCI offers CI/CD as a service to provide automation and end-to-end development workflows. Using CircleCI is similar to using GitHub Actions in the following ways:

* Configuration files for workflows are authored with YAML.
* Configuration files for workflows are stored with the repository code.
* Workflows include jobs.
* Jobs include steps that run sequentially.
* Steps can be shared with a community and reused.
* Variables can be set in the configuration file.
* Secrets can be created in the UI.
* Methods are provided to manually cache files via a configuration file.

A portion of a CircleCI pipeline that builds using Java and Gradle is shown in the next listing:

```yml
version: 2
jobs:
  build:
    environment:
      _JAVA_OPTIONS: "-Xmx3g"
      GRADLE_OPTS: "-Dorg.gradle.daemon=false -Dorg.gradle.workers.max=2"
    docker:
      - image: circleci/openjdk:11.0.3-jdk-stretch
    steps:
      - checkout
      - restore_cache:
          key: v1-gradle-wrapper-{{ checksum "gradle/wrapper/gradle-wrapper.properties" }}
      - restore_cache:
          key: v1-gradle-cache-{{ checksum "build.gradle" }}
      - run:
          name: Install dependencies
          command: "./gradlew build -x test"
      - save_cache:
          paths:
            - "~/.gradle/wrapper"
          key: v1-gradle-wrapper-{{ checksum "gradle/wrapper/gradle-wrapper.properties" }}
      - save_cache:
          paths:
            - "~/.gradle/caches"
          key: v1-gradle-cache-{{ checksum "build.gradle" }}
      - persist_to_workspace:
          root: "."
          paths:
            - build
```

There are also key differences between CircleCI and GitHub Actions, summarized in [Table 14-2](#key-differences-betw2).

Table 14-2. Key differences between CircleCI pipelines and GitHub Actions workflows

| Category | CircleCI pipelines | GitHub Actions workflows |
| --- | --- | --- |
| Test parallelism | Groups tests per custom rules or historical timing | Can use matrix strategy |
| Grouping multiple workflows | Declared as group in *config.yml* file with separate file for each workflow | Declared in individual workflow YAML files—no grouping |
| Use of Docker images for common dependencies | Provides prebuilt images with common dependencies with *USER* set to *circleci* | Uses actions as best approach to install dependencies |
| Caching | Provides *Docker Layer Caching* | Caching provided in common use cases |
| Specifying containers | First image listed in *config.yaml* is primary image used to run commands | Requires explicit section *container* for primary and *services* for additional containers |
| Reusable components | orbs | actions |

The listing that follows shows an example of converting the previous CircleCI pipeline portion to a comparable GitHub action as per the *actions-importer tool* (discussed later in this chapter) labs:

```yml
name: actions-importer-labs/circleci-demo-java-spring/workflow
on:
  push:
    branches:
    - main
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: openjdk:11.0.3-jdk-stretch
    env:
      _JAVA_OPTIONS: "-Xmx3g"
      GRADLE_OPTS: "-Dorg.gradle.daemon=false -Dorg.gradle.workers.max=2"
    steps:
    - uses: actions/checkout@v4.4.0
    - name: restore_cache
      uses: actions/cache@v4.3.1
      with:
        key: v1-gradle-wrapper-{{ checksum "gradle/wrapper/gradle-wrapper.properties" }}
        path: "~/.gradle/wrapper"
    - name: restore_cache
      uses: actions/cache@v4.3.1
      with:
        key: v1-gradle-cache-{{ checksum "build.gradle" }}
        path: "~/.gradle/caches"
    - name: Install dependencies
      run: "./gradlew build -x test"
    - uses: actions/upload-artifact@v4.1.1
      with:
        path: "./build"
```

More details on converting from CircleCI to GitHub Actions are available in the [documentation](https://oreil.ly/hi6Kl).

# GitLab CI/CD

GitLab is a hosting and software development platform similar to GitHub but is intended for use on-prem. Using it is similar to using GitHub Actions in the following ways:

* Configuration files for workflows are authored with YAML.
* Configuration files for workflows are stored with the repository code.
* Workflows include jobs.
* Jobs include steps that run sequentially.
* Jobs run on separate VMs or in separate containers.
* Jobs run in parallel by default but can be sequenced.
* Support is provided for setting variables in a configuration file.
* Support is provided for creating secrets in the UI.
* Methods are provided to manually cache files via a configuration file.
* Methods are provided to upload files and directories and persist them as artifacts.

An example GitLab CI/CD pipeline listing taken from the importer labs is shown here:

```yml
image: node:latest

services:
  - mysql:latest
  - redis:latest
  - postgres:latest

cache:
  paths:
    - node_modules/

test:
  script:
    - npm install
    - npm test
```

There are also key differences between GitLab CI/CD pipelines and GitHub Actions workflows, as shown in [Table 14-3](#key-differences-betw3).

Table 14-3. Key differences between GitLab CI/CD pipelines and GitHub Actions workflows

| Category | GitLab CI/CD | GitHub Actions |
| --- | --- | --- |
| Editor | GUI editor | YAML spec |
| Project designations | pipelines | workflows |
| Job platform identification keywords | *tags* | *runs-on* |
| Docker image identification keywords | *image* | *container* |
| Script step keywords | *script* | *run* (with *shell* if needed for particular shell) |
| Conditional keyword | *rules* | *if* |
| Sequential execution keyword | grouping via *stages* | *needs* |
| Scheduling workflows | scheduled via UI | *on:* keyword |
| Containers keyword | *image* | *container* |

The following listing shows an example of converting the previous GitLab CI pipeline to a comparable GitHub Actions workflow:

```yml
name: actions-importer/node-example
on:
  push:
  workflow_dispatch:
concurrency:
  group: "${{ github.ref }}"
  cancel-in-progress: true
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: node:latest
    timeout-minutes: 60
    services:
      mysql:latest:
        image: mysql:latest
      redis:latest:
        image: redis:latest
      postgres:latest:
        image: postgres:latest
    steps:
    - uses: actions/checkout@v4.4.0
      with:
        fetch-depth: 20
        lfs: true
    - uses: actions/cache@v4.3.1
      with:
        path: node_modules/
        key: default
    - run: npm install
    - run: npm test
```

More details on converting from GitLab CI/CD to GitHub Actions can be found in the [documentation](https://oreil.ly/Och9c).

# Jenkins

Jenkins was one of the earliest CI/CD orchestration tools. Like GitHub Actions, it offers a comprehensive engine to automate, monitor, and facilitate continuous pipelines as well as other types of automation jobs. It is supported by an extensive set of plug-ins and has a *pipeline* type of project to enable coding software pipelines. Using it is similar to using GitHub Actions in the following ways:

* Configuration and workflows are created via declarative pipelines, which are similar to GitHub Actions.
* Configuration files for workflows can be stored with the repository code.
* Jenkins pipelines use stages to group steps together, similar to GitHub Actions’ use of jobs.
* Collections of steps can run on separate VMs or in separate containers.
* Plug-ins in Jenkins are similar to actions for GitHub Actions.
* Jenkins allows sending builds to one or more build *agents*, similar to GitHub *runners*.
* Jenkins allows for defining a matrix of various system combinations to run against.

Here is an example Jenkins pipeline listing from the importer labs sample:

```conf
pipeline {
    agent {
        label 'TeamARunner'
    }

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Check Variables') {
            steps {
                echo "Database engine is ${DB_ENGINE}"
                echo "DISABLE_AUTH is ${DISABLE_AUTH}"
            }
        }
        stage('Build') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Test') {
            steps {
                junit '**/target/*.xml'
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```

There are also key differences between Jenkins pipelines and GitHub Actions workflows, summarized in [Table 14-4](#key-differences-betw4).

Table 14-4.
Key differences between Jenkins declarative pipelines and GitHub Actions workflows

| Category | Jenkins declarative pipelines | GitHub Actions workflows |
| --- | --- | --- |
| Workflow format | Declarative pipelines | YAML spec |
| Executor keyword | *agent* | *runner* |
| Tool access | *tools* keyword | *installed on runner* |
| Grouping syntax | Groups steps together via *stages* | Groups steps together via *jobs* |
| Environment settings | *environment* | *.env* with a job or step id |
| Strategy specification | *options* | *strategy* with a job id |
| Input/output specification | *parameters* | *inputs/outputs* |
| Execution on a schedule | Jenkins cron syntax | *on.schedule* |
| Conditional execution keyword | *when* | *if* |
| Parallel execution | *parallel* keyword | Runs in parallel by default |

The listing that follows shows an example of converting the previous Jenkins pipeline to a comparable GitHub action:

```yml
name: demo_pipeline
on:
  workflow_dispatch:
env:
  DISABLE_AUTH: 'true'
  DB_ENGINE: sqlite
jobs:
  Check_Variables:
    name: Check Variables
    runs-on:
      - self-hosted
      - TeamARunner
    steps:
    - name: checkout
      uses: actions/checkout@v4.4.0
    - name: echo message
      run: echo "Database engine is ${{ env.DB_ENGINE }}"
    - name: echo message
      run: echo "DISABLE_AUTH is ${{ env.DISABLE_AUTH }}"
  Build:
    runs-on:
      - self-hosted
      - TeamARunner
    needs: Check_Variables
    steps:
    - name: checkout
      uses: actions/checkout@v4.4.0
    - name: Upload Artifacts
      uses: actions/upload-artifact@v4.1.1
      if: always()
      with:
        path: "**/target/*.jar"
  Test:
    runs-on:
      - self-hosted
      - TeamARunner
    needs: Build
    steps:
    - name: checkout
      uses: actions/checkout@v4.4.0
    - name: Publish test results
      uses: EnricoMi/publish-unit-test-result-action@v2.19.0
      if: always()
      with:
        junit_files: "**/target/*.xml"
  Deploy:
    runs-on:
      - self-hosted
      - TeamARunner
    needs: Test
    steps:
    - name: checkout
      uses: actions/checkout@v4.4.0
    - name: sh
      shell: bash
      run: make publish
```

More details on converting from Jenkins to GitHub Actions can be found in the [documentation](https://oreil.ly/42e5d).

# Travis CI

Travis CI is a hosted continuous integration service that can build, test, and automate software delivery. It provides services to software projects in various hosting platforms. Using it is similar to using GitHub Actions in the following ways:

* Configuration files for workflows are authored with YAML.
* Configuration files for workflows are stored with the repository code.
* It lets you manually cache dependencies for later reuse.
* It supports using a matrix for performing testing.
* Status badges can be created to display build pass/fail info.
* It supports parallelism.

A simple Travis CI example is shown here (taken from the importer labs example):

```yml
language: ruby
dist: trusty
rvm:
- 1.9.3
- 2.0.0
- 2.1.0

install:
- gem install bundler

script:
- echo "Processing"

jobs:
  include:
    - script: echo "sub-processing"
```

There are also key differences between Travis CI and GitHub Actions workflows, summarized in [Table 14-5](#key-differences-betw5).

Table 14-5. Key differences between Travis CI pipelines and GitHub Actions workflows

| Category | Travis CI pipelines | GitHub Actions workflows |
| --- | --- | --- |
| Reusable components | *phases* | *jobs* |
| Target specific branches | *branches: only:* | *on: push: branches;* |
| Parallel execution construct | *stages* | *jobs* |
| Script step keyword | *script* | *run* (with shell if needed for particular shell) |
| Matrix specifications | *matrix: include* | *jobs: build: strategy: matrix:* |

The following listing shows an example of converting the previous Travis CI pipeline into a comparable GitHub Actions workflow:

```yml
name: travisci-ruby-example
on:
  push:
    branches:
    - "**/*"
  pull_request:
jobs:
  primary:
    runs-on: ubuntu-latest
    steps:
    - name: checkout
      uses: actions/checkout@v4.5.0
    - uses: ruby/setup-ruby@v1.237.0
      with:
        ruby-version: "${{ matrix.rvm }}"
    - run: gem install bundler
    - run: echo "Processing"
    strategy:
      matrix:
        rvm:
        - 1.9.3
        - 2.0.0
        - 2.1.0
        - 3.3.0
  secondary:
    runs-on:
             ubuntu-latest
    steps:
    - name: checkout
      uses: actions/checkout@v4.5.0
    - uses: ruby/setup-ruby@v1.237.0
      with:
        ruby-version: 3.3
    - run: gem install bundler
    - run: echo "sub-processing"
```

More details on converting from Travis CI to GitHub Actions can be found in the [documentation](https://oreil.ly/8mzoh).

# GitHub Actions Importer

As shown by the examples already used in this chapter, migrating one workflow, or even a few workflows, is usually not too difficult. However, what do you do when you have hundreds, or even thousands, of migrations to do? For such cases, as well as bootstrapping migrations, GitHub has provided a tool called the *GitHub Actions Importer* to help import pipelines from other CI/CD platforms to GitHub Actions workflows. With this tool, you can get part of the way through a simple migration automatically. But, as you would expect, the tool does not guarantee completeness or correctness on any given migration.

The tool is designed to import from the same six CI/CD platforms as discussed elsewhere in this chapter:

* Azure DevOps
* Bamboo
* CircleCI
* GitLab
* Jenkins
* Travis CI

The tool is provided via a Docker container that is run as an extension to the GitHub CLI. Thus, to use it, you will need to have Docker installed and running as well as the official [GitHub CLI](https://cli.github.com).

# Docker and Windows

If you are running on Windows, Docker must be configured to use Linux-based containers.

With the GitHub CLI installed, you can install the *actions importer extension* via the following command:

```sh
gh extension install github/gh-actions-importer
```

The importer supplies a number of commands that can be used in the migration process. These are summarized in [Table 14-6](#github-actions-import).

Table 14-6. GitHub Actions Importer commands

| Command | Function |
| --- | --- |
| update | Update to the latest version of GitHub Actions Importer |
| version | Display the current version of GitHub Actions Importer |
| configure | Start an interactive prompt to configure credentials used to authenticate with your CI server(s) |
| audit | Plan your CI/CD migration by analyzing your current CI/CD footprint |
| forecast | Forecast GitHub Actions usage from historical pipeline utilization |
| dry-run | Convert a pipeline to a GitHub Actions workflow and output its YAML file |
| migrate | Convert a pipeline to a GitHub Actions workflow and open a pull request with the changes |

# Updating the Importer CLI

To ensure you’re using the latest version of the importer tool, it’s a good idea to regularly run the `update` command:

```sh
gh actions-importer update
```

Generally, a phased approach works well for doing migrations. The recommended phases are as follows:

1. Planning for when you want to do the migration and estimating the complexity via an audit
2. Understanding your current compute usage via forecasting
3. Doing a dry run of the conversion
4. Performing the production workflow migration

I’ll cover these phases in individual sections next. But in order to use the importer for any phase, you will first need to configure credentials for it and then authenticate it via the `configure` command.

## Authentication

To do its work, the importer tool has to have access both to the platform you are converting from and, of course, to the target repo(s) in GitHub. This means you need to be able to supply it with a few pieces of data for authentication, including the following:

* A GitHub personal access token
* The URL for the GitHub instance you are using
* An access token from the platform you are converting from
* The location (URL) of the running application instance that you are converting from
* A username that has access on the platform you are converting from

While you can put these into environment variables, the importer tool offers the `configure` command as an interactive way to set these. Here is the basic form of the command:

```sh
gh actions-importer configure
```

After running this, you’ll first be prompted to select the platform you’re converting from. You can just use the arrow key and space to select one. Then hit Enter. Afterward, you’ll be prompted to interactively supply the other data inputs. Here’s an example run for working with a Jenkins instance:

```yml
gh actions-importer configure
✔ Which CI providers are you configuring?: Jenkins
Enter the following values (leave empty to omit):
✔ Personal access token for GitHub: *******************************
✔ Base url of the GitHub instance: https://github.com
✔ Personal access token for Jenkins: *******************************
✔ Username of Jenkins user: admin
✔ Base url of the Jenkins instance: http://localhost:8080/
Environment variables successfully updated.
```

Upon completion of the command, the importer tool will write the data to an *.env.local* file. This file can be populated in advance or individual environment variables set if you prefer that way of supplying the data needed for configuration:

```sh
cat .env.local
GITHUB_ACCESS_TOKEN=ghp_P73jshbAcUmCQvaOyAIuxNUE---------
GITHUB_INSTANCE_URL=https://github.com
JENKINS_ACCESS_TOKEN=117e5929321809d5eeb9a91684--------
JENKINS_INSTANCE_URL=http://localhost:8080/
JENKINS_USERNAME=admin
```

After running through the initial configuration for the importer, you’re ready to move on to the other phases, such as auditing to help with planning your migration.

## Planning

Planning is a required first step in doing migrations to GitHub Actions. You have to understand where you’re starting from and where you want to go to figure out how to get there. The following are the kinds of questions you should be thinking about at this point:

* Which pipelines should be migrated?
* How customized are these pipelines?
* Should these pipelines be *refactored* before migration?
* What kind of compute and runtime environments are used/needed?

The Actions Importer tool provides the *audit* command to help analyze the complexity of migrating your pipelines and help create a migration plan. The purpose of this command is to gather all of the pipelines scoped for a migration, try and run a conversion, and then produce a summary report with statistics based off of the attempted conversions.

To run an audit with the tool, you would use a command like the following:

```sh
gh actions-importer audit jenkins --output-dir tmp/audit
```

The resulting report provides aggregated details from the levels of a pipeline and the steps within it. It also flags the migration tasks that cannot be automatically completed and will need manual intervention.

An example of the *Pipelines* summary section is shown in [Figure 14-2](#example-audit-summary).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1402.png)<br>
**Figure 14-2. Example audit summary pipelines section**

The metrics provided by the *Pipelines* section are summarized in [Table 14-7](#metrics-from-pipeline).

Table 14-7. Metrics from Pipelines section of report

| Metric | Meaning |
| --- | --- |
| Total | Total number of pipelines processed. |
| Successful | Count of pipelines that could be completely automatically converted. |
| Partially successful | Count of pipelines that had all constructs converted but some individual items that could not be converted. |
| Unsupported | Count of pipelines that use constructs not supported by GitHub Actions Importer. |
| Failed | Count of pipelines with a fatal error during conversion attempt. Could be because of a bad original pipeline, internal error with importer, invalid credentials, network error, etc. |

There is also a *Job types* part included in the *Pipelines* section. This part provides a summary of which pipeline types are being used and whether they are supported or unsupported by the importer. The information here will vary based on the CI/CD platform the import is based on.

As an example of the type of data in this part, Jenkins job types might be broken down into categories like *WorkflowMultiBranchProject* for Jenkins jobs that are of the *Multi-branch pipeline* type in Jenkins. Also, Jenkins allows for pipelines to be written in *scripted* syntax versus *declarative* syntax. While work is being done to support scripted pipelines in Jenkins, as of the time of this writing, the number of Jenkins jobs written in scripted syntax would show up in the *Unsupported* list.

## Build Steps and Related

Going down a level further, in this section of the audit report, the importer tool provides an aggregated summary for the individual build steps and the ability (or not) to convert them automatically.

An example of this section is shown in [Figure 14-3](#build-steps-section-o).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1403.png)<br>
**Figure 14-3. Build steps section of audit output**

The metrics provided by the Build steps section are listed in [Table 14-8](#metrics-provided-by-b).

Table 14-8. Metrics provided by Build steps section of report

| Metric | Meaning |
| --- | --- |
| Total | Total number of build steps processed |
| Known | Provides a breakdown by type of build steps that can be automatically converted to an action |
| Unknown | Provides a breakdown by type of build steps that cannot be automatically converted to an equivalent action |
| Unsupported | Provides a breakdown by type of build steps that are either not supported by GitHub Actions or configured in a way that is not compatible with GitHub Actions |
| Actions | Provides a breakdown of actions that would be used in converted workflows |

There are also several other miscellaneous sections between the *build* and *manual steps* sections. These include *build triggers*, *environment variables*, and other constructs (if there are any) that don’t fit into any of the categories. An example of the output for this section is shown in [Figure 14-4](#triggers-environment).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1404.png)<br>
**Figure 14-4. Triggers, environment variables, etc., identified from the audit**

## Manual Tasks

The report also contains a section titled *Manual tasks* that identifies tasks that someone will need to handle manually at a repository/system level to ensure the workflow can function in a GitHub Actions environment. There are two primary types of items reported here:

> *Secrets*
>> This is a list of secrets used in the pipelines that are converted. Since secrets have to be set up separately, they need to be created manually for the repositories.
>
> *Self-hosted runners*
>> This is a list of the labeled runners or build agents that were identified in the pipelines that were converted. These will have to be handled via GitHub-hosted runners or self-hosted runners in GitHub Actions.

Example output for manual tasks identified during an audit is shown in [Figure 14-5](#manual-tasks-identifi).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1405.png)<br>
**Figure 14-5. Manual tasks identified during an audit**

# Other Things That May Not Be Migrated Automatically

In addition to secrets, encrypted values, and build agents that are not converted automatically, there are several other constructs that will need manual follow-up:

> *Packages*
>> Packages referenced/created in the original pipeline are not migrated to GitHub packages. Steps that publish or consume artifacts and caches are converted.
>
> *Permissions*
>> Permissions and credentials for pipelines are not migrated automatically because they will need to be set up on the target system.
>
> *Triggers*
>> Types of events that trigger builds may not be converted automatically.

## File Manifest

The last section of the audit report provides a list of files that were generated during the audit. This list may include the following:

* The original pipeline specification file(s) as defined in the original CI/CD platform
* Log of network responses during the pipeline conversion
* Converted workflow file(s)
* Log of error messages to help debug/troubleshoot any failed pipeline conversions

Example output for the files manifest from an audit is shown in [Figure 14-6](#files-manifest-from-a).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1406.png)<br>
**Figure 14-6. Files manifest from an audit run**

## Forecasting

The *forecast* command looks at jobs that have been completed over a period of time and attempts to calculate usage metrics from that data. The intent is to assist in figuring out how much compute capacity you’re using in your current environment so you can plan for what you’ll need when you convert to GitHub Actions.

An example forecast command could look like this:

```sh
gh actions-importer forecast jenkins --output-dir tmp/forecast
--start-date 2022-08-02
```

# Supplying a Date for the Actions-Importer Command

When running an actions-importer command that takes a date, you need to supply a date that was prior to when the jobs of interest were seeded, or at least one that will allow the tool to capture a period of typical usage. The default value is one week ago.

The metrics are provided for each runner queue in your current system. A brief summary of the metrics that come out of running forecasting (relative to the *date* parameter) is in [Table 14-9](#metrics-from-forecast).

Table 14-9. Metrics from Forecast section of the report

| Metric | Purpose |
| --- | --- |
| Job count | Total number of completed jobs |
| Pipeline count | Number of unique pipelines used |
| Execution time | Amount of time a runner spent on a job |
| Queue time | Amount of time spent waiting for a runner to be available |
| Concurrent jobs | Number of jobs running at any given point in time |

The execution time and concurrent jobs metrics can be used to create estimates of the cost and concurrency, respectively, of GitHub Actions runners that will be needed after your conversion.

An example forecast report output is shown in [Figure 14-7](#example-forecast-repo).

There is a section of the report that follows the *Total* section with a similar format. That section is a breakdown of consumption metrics aggregated by queues of runners. This is useful if you have runners on different platforms (Windows, Mac, Linux) and want to see how much each platform was utilized along with corresponding metrics. If you see an *N/A* here at the top of that section, that is because your server didn’t have any additional runners/agents in use.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1407.png)<br>
**Figure 14-7. Example forecast report output**

## Doing a Dry Run

Before you do the actual migration of your pipeline, you should do a test conversion (a *dry run*). The *dry-run* command of the actions importer shows you what the results would be of converting a pipeline to the GitHub Actions equivalent workflow. The output from it shows the files that were generated. Logs are written to an output directory.

An example dry-run command might look like the following:

```yml
gh actions-importer dry-run jenkins --source-url
http://localhost:8080/job/test_pipeline --output-dir tmp/dry-run
```

The dry-run command will result in a GitHub Actions pipeline being created in the *tmp/dry-run/<job-name>/.github/workflows/<job-name>.yml* area. The importer process may not be able to automatically convert every section/construct of your pipeline. For those cases where it cannot do the conversion, the parts that could not be converted will be commented out in the resulting workflow. An example is shown in the commented section of the next listing:

```yml
jobs:
  build:
    runs-on:
      - self-hosted
      - TeamARunner
    steps:
    - name: checkout
      uses: actions/checkout@v4.3.0
    - name: echo message
      run: echo "Database engine is ${{ env.DB_ENGINE }}"
#     # This item has no matching transformer
#     - sleep:
#       - key: time
#         value:
#           isLiteral: true
#           value: 80
    - name: echo message
      run: echo "DISABLE_AUTH is ${{ env.DISABLE_AUTH }}"
  test:
    runs-on:
      - self-hosted
```

In those cases, either you can edit the converted workflow file manually to resolve the issue or you can implement a *custom transformer* that you can provide to the importer to address the issue. The custom transformer is useful if you have the same issue in multiple pipelines.

## Creating Custom Transformers for the Importer

A custom transformer allows you to tell the transformer how to convert some part of your pipeline that the importer doesn’t handle automatically. The transformer can be implemented via a file written in the *Ruby* programming language. A simple example (taken from the [labs document for the actions-importer project](https://oreil.ly/j-O-k)) follows.

Assume you have done a dry run that results in a section of code that could not be automatically converted, as in the previous listing. The section of code that could not be automatically converted is denoted by the comment markers that the importer added:

```yml
#     # This item has no matching transformer
#     - sleep:
#       - key: time
#         value:
#           isLiteral: true
#           value: 80
```

For a GitHub Actions workflow, that code can be replaced by a simple shell command implemented in the form of a workflow step, like the following:

```yml
- name: Sleep for 80 seconds
  run: sleep 80s
  shell: bash
```

To teach the importer tool how to do the conversion automatically, you could write a custom transformer using Ruby and workflow syntax. Here’s an example of what that code might look like:

```yml
transform "sleep" do |item|
  wait_time = item["arguments"][0]["value"]["value"]

  {
    "name": "Sleep for #{wait_time} seconds",
    "run": "sleep #{wait_time}s",
    "shell": "bash"
  }
```

Since it is written in Ruby, custom transformers can have any valid Ruby syntax. The main point is that it needs to return a hash that has the YAML to be generated for the given step. The *item* parameter is used to get the needed values from the original code. In this case, you can map the following:

```yml
wait_time = item["arguments"][0]["value"]["value"]
```

to the original *value.value* syntax:

```yml
#         value:
#           isLiteral: true
#           value: 80
```

This code can then be stored in a file with an *.rb* extension and provided to the importer on the command line via the *--custom-transformers* option. Here’s an example of what that command could look like (assuming the preceding code is saved in a file *transformer1.rb*):

```sh
gh actions-importer dry-run jenkins
 --source-url http://localhost:8080/job/test_pipeline
 --output-dir tmp/dry-run --custom-transformers transformer1.rb
```

After running this command, the file in *tmp/dry-run/test_pipeline/.github/workflows/test_pipeline.yml* will include the results of the transformer being applied. The updated section is shown in this listing:

```yml
  steps:
    - name: checkout
      uses: actions/checkout@v4.3.0
    - name: echo message
      run: echo "Database engine is ${{ env.DB_ENGINE }}"
    - name: Sleep for 80 seconds
      run: sleep 80s
      shell: bash
    - name: echo message
      run: echo "DISABLE_AUTH is ${{ env.DISABLE_AUTH }}"
```

### A more detailed example

To better understand how to go about deciding on a custom transformer approach, here’s another, more detailed example. Suppose that you have the following code in your Jenkins pipeline:

```groovy
stage('write-results') {
    steps {
        writeFile file: 'results.out', text: 'These are the results.'
    }
}
```

When you go through the *dry-run* command in the actions-importer, it produces output showing that it can’t convert that code, as per the commented section in the next listing:

```yml
  write_results:
    name: write-results
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: checkout
      uses: actions/checkout@v4.3.0
#     # This item has no matching transformer
#     - writeFile:
#       - key: file
#         value:
#           isLiteral: true
#           value: results.out
#       - key: text
#         value:
#           isLiteral: true
#           value: These are the results.
```

Assume that you investigate and determine that, based on the intent to write something to a file, the following would be a good substitute for your converted workflow:

```yml
uses: DamianReeves/write-file-action@v1.3
with:
  path: ${{ env.home}}/results.out
  contents: |
    These are the results.
  write-mode: append
```

This is a bit more complex than the simple *sleep* example presented previously. You need to understand more about how to map the arguments appropriately between the Jenkins construct and the GitHub action. You can determine more information about the Jenkins construct *writeFile* by writing a simple transformer to print out info about its arguments and its mapping. You do this by using the Ruby print equivalent *puts* and passing in the name of the construct. Here’s an example transformer that does that processing to get the information:

```ruby
transform "writeFile" do |item|
  puts "This is the item: #{item}"
end
```

After storing this file as *transformer2.rb,* you can do a dry run with it to see the printed output:

```sh
gh actions-importer dry-run jenkins --source-url http://localhost:8080/job/test_pipeline --output-dir tmp/dry-run
--custom-transformers transformers/transformer2.rb
[2023-03-18 13:43:19]
Logs: 'tmp/dry-run/log/valet-20230318-134319.log'
This is the item: {"name"=>"writeFile", "arguments"=>
[{"key"=>"file", "value"=>{"isLiteral"=>true,
"value"=>"results.out"}}, {"key"=>"text",
"value"=>{"isLiteral"=>true,
"value"=>"These are the results."}}]}
[2023-03-18 13:43:19] Output file(s):
[2023-03-18 13:43:19]
tmp/dry-run/test_pipeline/.github/workflows/test_pipeline.yml
```

What this provides you is the mapping for the Jenkins construct. You can then use this to write a custom transformer to map the argument values from the Jenkins construct to the argument values for the GitHub action call.

Given this particular output, you could create a custom transformer that looks like this:

```ruby
transform "writeFile" do |item|
  file_arg = item["arguments"].find{ |a| a["key"] == "file" }
  file_path = file_arg.dig("value", "value")
  text_arg = item["arguments"].find{ |a| a["key"] == "text" }
  text = text_arg.dig("value", "value")
  {
    "uses" => "DamianReeves/write-file-action@v1.2",
    "with" => {
      "path" => "${{ env.home}}//"+file_path,
      "contents" => text
     }
  }
end
```

The *file_arg* and *file_path* variables get the hashes associated with each of the arguments for the Jenkins construct. The Ruby *dig* command is then used to extract the values from the hashes into the *file_path* and *text_arg* variables, respectively. Then those variables are simply plugged into the general transformer substitution.

Doing a dry run with this transformer results in the following workflow code being automatically generated:

```yml
  write_results:
    name: write-results
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: checkout
      uses: actions/checkout@v4.3.0
    - uses: DamianReeves/write-file-action@v1.3
      with:
        path: "${{ env.home}}//results.out"
        contents: These are the results.
```

### Other transformers

In addition to custom transformers for constructs like steps and stages, you can also add simple replacement transformers to update values for environments and runner systems.

The idea is not that you are telling the importer how to convert these into workflow code. Rather, you are simply telling it to change the value in the results. These are straightforward and don’t require coding. You simply include them at the top of a custom transformer file. Examples of both types are shown here.

To set the value of the *CURRENT_LEVEL* environment variable to *dev* versus what it was in the original Jenkinsfile, you could use the following:

```yml
env "CURRENT_LEVEL", "dev"
```

Likewise, to change the value of a runner:

```yml
runner "mynode", "self-hosted"
```

There’s one final note on the mechanics of pulling in transformers. You can point the *custom-transformers* option to an individual file containing transformer code, or you can use multiple files or glob patterns. The following are examples of valid commands:

```sh
gh actions-importer dry-run jenkins —source-url $YOUR_SOURCE_URL -o output —custom-transformers transformers1.rb transformers2.rb

gh actions-importer dry-run jenkins —source-url $YOUR_SOURCE_URL -o output —custom-transformers transformers/*.rb
```

## Doing the Actual Migration

The importer’s *migrate* command will fetch a pipeline’s code, convert it to the equivalent GitHub Actions workflow, and then open a pull request to a repository with the converted workflow.

Prior to running the actual migration, you should have completed any auditing activities with the importer and have also done a *dry run*. The migrate command expects the same sort of parameters as discussed for the other importer steps:

* The source URL of the item you want to convert
* Where you want to store the logs
* The URL for the GitHub repo to put the resulting workflow in

# Target GitHub Repo

Prior to running the migrate command, you should ensure the target repo has been created/exists.

Here is a possible example of running a migrate command (using my destination repository and a version of the pipeline that was used in the dry-run section):

```sh
gh actions-importer migrate jenkins \
--target-url https://github.com/importer-test/prod_pipeline \
--output-dir tmp/migrate \
--source-url http://localhost:8080/job/prod_pipeline \
--custom-transformers jenkins/transformers/transformer1.rb
```

Notice that I’m also including the same custom-transformers as worked out during the *dry-run* phase. The output from the command is straightforward but includes a very important link that you will need for next steps—the link to a new pull request:

```log
[2023-03-21 11:26:27] Logs:
'tmp/migrate/log/valet-20230321-112627.log'
[2023-03-21 11:26:29] Pull request:
'https://github.com/importer-test/prod_pipeline/pull/2'
```

When the migration command is run, if successful, it will finish by creating a pull request in the target repository with the converted code. [Figure 14-8](#example-pull-request) shows an example pull request from a migrate command.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1408.png)<br>
**Figure 14-8. Example pull request from importer migrate run**

One of the benefits of having the *migrate* step generate a pull request is that you can review the code that was produced or run any workflows for pull requests against it to ensure it is correct before merging. If a *migrate* run results in code that isn’t how you want it, you can always update things with methods like transformers and do another *migrate* run. This will simply produce another pull request from the updated processing. [Figure 14-9](#reviewing-code-in-pu2) shows an example of reviewing the converted code produced by one of the transformers used in the previous *dry-run* discussion.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1409.png)<br>
**Figure 14-9. Reviewing code in pull request from migrate run**

If the code you are migrating includes references to settings or pieces that are not defined within your pipeline, the pull request will include a *Manual steps* section listing what you need to do manually to complete the conversion. For example, in my pipeline that I’m targeting for migration, assume I’m referencing specific credentials that had only been defined within my Jenkins instance as follows:

```yml
    stages {
        stage('build') {
            steps {
              withCredentials([usernamePassword(credentialsId: 'build-admin', passwordVariable: 'USER_PASS', usernameVariable: 'USER_NAME')]) {
                echo "Building..."
```

In this case, the pull request would include a Manual steps section, as shown in [Figure 14-10](#pull-request-from-mig).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1410.png)<br>
**Figure 14-10. Pull request from migrate with manual steps to be completed**

Once the manual steps and any code reviews of the pull request are completed, the pull request can be merged, and the workflow will have been successfully migrated.

# Enabling Self-Service Migrations via IssueOps

As another option to downloading and running the importer command, you can run the importer via GitHub Actions and GitHub Issues. For more details, see [the GitHub Actions Importer Issue Ops repository](https://oreil.ly/p_ass).

# Conclusion

GitHub has long supported integration with multiple, different automation platforms. With GitHub Actions, users can now choose instead to more tightly integrate with automation built into GitHub—after migrating from the old platform to GitHub Actions.

In this chapter, we’ve covered migration approaches at a high level, highlighting similarities and some key differences to have in mind before starting the process. Being prepared and mapping out plans of attack around the core areas of source, processing, users, and infrastructure are key to achieving a successful migration.

The GitHub Actions Importer tool can assist in doing a migration in many cases. The tooling includes functionality to assess and forecast impacts of migrating as well as enabling dry runs to try out automation before actually making the changes. The importer understands how to work with pipelines from Azure DevOps, CircleCI, GitLab CI/CD, Jenkins, Travis CI, and Bamboo.

In some cases, the importer may not be able to translate from one of the other platform constructs to a corresponding GitHub Actions construct. Once the user determines the best way to make the translation, the changes can be made manually in the new workflow, or if the same translation needs to be repeated multiple times, a custom transformer can be written and pulled in when running the importer.

While migrating from another CI/CD integration platform to GitHub Actions may seem like a daunting task, many constructs and processes in the other platforms have corresponding implementations in Actions. These conventions can simplify the conversion once the correct categories such as jobs, stages, steps, and keywords are identified. When neither a pass at manually converting nor using the importer tool seems to yield the best results, a hybrid approach of running through the importer tool first to get a starting point and then manually adjusting the resulting workflow may provide the best option.

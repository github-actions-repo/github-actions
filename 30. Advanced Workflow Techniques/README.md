# Chapter 13. Advanced Workflow Techniques

There are a set of techniques that you can use in your workflows to greatly simplify processing for a couple of less-often-encountered use cases. In this chapter, I’ve collected a few of these so that you can have them as additional tools in your Actions toolbox.

In the first part of the chapter, we’ll discuss how you can interact with GitHub components and drive GitHub functions from within your workflow through a variety of interfaces.

Then, we’ll cover more about leveraging the *matrix strategy* in GitHub Actions to automatically spin up sets of jobs spanning multiple input dimensions. [Chapter 8](ch08.html#ch08) provided an intro to this, but there’s much more to discuss and explore.

Finally, we’ll cover the multiple ways you can use containers as a technique to encapsulate different environments, technologies, and functionality for your workflow to use. There are several versatile ways to take advantage of containers in your workflows, and I’ll explain each with examples. Note that this is different from building a container action as described in [Chapter 11](ch11.html#ch11).

As a starting point, here’s how you can tie in more directly with GitHub operations from inside of your workflows.

# Driving GitHub from Your Workflow

Sometimes you may need or want to do more with GitHub components than the usual structures and flows will allow. Of course, you could create a custom action to handle the task (per [Chapter 11](ch11.html#ch11)), but that may be overkill if you only need to do a task one time or can do it within a single step.

In this section, we’ll look at some techniques you can leverage within your workflow to drive lower-level GitHub functionality. Topics in this area include the following:

* Using the GitHub CLI
* Creating scripts
* Invoking GitHub APIs

Up first is accessing the GitHub command line interface (CLI) from within your workflows.

## Using the GitHub CLI

GitHub provides a simple CLI that can be downloaded and used for various GitHub-specific operations and entities, including pull requests, issues, repos, releases, etc. The executable for this is named `gh`.

If you are executing your workflows on a GitHub-hosted runner, the GitHub CLI is already installed and available for your use. To incorporate it in a workflow, you can simply call the `gh` executable from a `run` command in a step. The only prerequisite for this is that you need to set an *environment variable* named GITHUB_TOKEN to a *token* with the required access and scope to run the CLI. Normally, simply setting the environment variable to the GITHUB_TOKEN from the `secrets` context should suffice.

An example of using the CLI to create a new GitHub issue in a reusable workflow is shown in the next listing:

```yml
name: create issue via gh

on:
  workflow_call:
    inputs:
      title:
        description: 'Issue title'
        required: true
        type: string
      body:
        description: 'Issue body'
        required: true
        type: string
    outputs:
      issue-number:
        description: "The issue number"
        value: ${{ jobs.create-issue.outputs.inum }}

jobs:
  create-issue:
    runs-on: ubuntu-latest
    # Map job outputs to step outputs
    outputs:
      inum: ${{ steps.new-issue.outputs.inum }}

    permissions:
      issues: write

    steps:
      - name: Create issue using CLI
        id: new-issue
        run: |
          response=`gh issue create \
          --title "${{ inputs.title }}" \
          --body "${{ inputs.body }}" \
          --repo ${{ github.event.repository.url }}`
          echo "inum=$(echo $response | rev | cut -d'/' -f 1 | rev)" >> $GITHUB_OUTPUT
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The main items to pay attention to in this listing are lines 34–37. This is a single shell command that calls the `gh` function to create a new issue. The issue is created with the inputs from the workflow call. Line 38 takes the output of the command and parses it to produce just the id of the new issue. Note that lines 39–40 pass the GITHUB_TOKEN in for the `gh` application to use.

If you need to accomplish more than what the CLI can provide, another option is using an action that allows you to script some GitHub calls.

## Creating Scripts

Suppose your workflow needs some lower-level GitHub functionality to accomplish something fairly simple. And, suppose it would be overkill to create something external to the workflow to encapsulate the functionality. One other option you have is writing a small program (or *script*) inline. You can do this by leveraging the [github-script action](https://oreil.ly/GYIDh). This action allows you to write a script in a workflow that has access to the GitHub API and the workflow `run` context.

To do this, you simply `use` the script action with an input named `script` that contains the body of the script. Here’s an example from the action’s documentation. The use case here is applying a label to an issue:

```yml
steps:
  - uses: actions/github-script@v6
    with:
      script: |
        github.rest.issues.addLabels({
          issue_number: context.issue.number,
          owner: context.repo.owner,
          repo: context.repo.repo,
          labels: ['Triage']
        })
```

When you use this action, you have access within the script to several of the *packages* available from the [actions toolkit](https://oreil.ly/Y7T99) without importing them. A basic description of those (taken from the doc) is shown in [Table 13-1](#available-packages-th). (You can find more information about the Actions Toolkit in [Chapter 11](ch11.html#ch11) on creating custom actions.)

Table 13-1. Available packages through the github-script plug-in

| Package/functionality | Description |
| --- | --- |
| github | Pre-authenticated [octokit/rest.js client](https://oreil.ly/15rn3) |
| context | [Context of the workflow run](https://oreil.ly/s5k1X) |
| core | Reference to the [@actions/core package](https://oreil.ly/-HEL5) |
| glob | Reference to the [@actions/glob package](https://oreil.ly/AZL6e) |
| io | Reference to the [@actions/io package](https://oreil.ly/pXev6) |
| exec | Reference to the [@actions/exe packagec](https://oreil.ly/Ft0Kz) |
| fetch | Reference to the [node-fetch package](https://oreil.ly/0SCVf) |
| require | Proxy wrapper around the normal Node.js *require*; paths are relative to current working directory |

You can find many more usage examples on the [actions page on Marketplace](https://oreil.ly/P2bFm). Alternatively, or if you need more direct access to the GitHub API, you can invoke that directly from your workflow as well.

## Invoking GitHub APIs

Besides the GitHub CLI or the script action, GitHub’s REST API can be used directly to do similar functions. You’ve seen an example of using the CLI to create an issue in a preceding section. So, I’ll repeat the job definition for that one but using the GitHub REST API invocation that corresponds to the CLI version:

```yml
create-issue:
  runs-on: ubuntu-latest
  # Map job outputs to step outputs
  outputs:
    inum: ${{ steps.new-issue.outputs.inum }}

  permissions:
    issues: write

  steps:
    - name: Create issue using REST API
      id: new-issue
      run: |
        response=$(curl --request POST \
          --url https://api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.PAT }}' \
          --header 'content-type: application/json' \
          --data '{
            "title": "${{ inputs.title }}",
            "body": "${{ inputs.body }}"
          }' \
          --fail | jq '.number')
        echo "inum=$response" >> $GITHUB_OUTPUT
```

The call, starting at line 13, follows standard REST API syntax invoked via the *curl* command. Notice that the header also includes a personal access token to provide the necessary authorization (line 15). Similar to the CLI example, the output from the command is parsed (via the *jq* tool) to pick out the number of the new issue.

The GitHub REST API can also be used to invoke another workflow. The following listing shows an example of using a REST API call to invoke a workflow stored as *create-failure-issue.yml* in the same repository (line 12):

```yml
create-issue-on-failure:
  runs-on: ubuntu-latest
  needs: test-run
  if: always() && failure()
  steps:
    - name: Invoke workflow to create issue
      run: |
        curl -X POST \
          -H "authorization: Bearer ${{ secrets.PIPELINE_USE }}" \
          -H "Accept: application/vnd.github.v3+json" \
          "https://api.github.com/repos/${{ github.repository }}/actions/workflows/create-failure-issue.yml/dispatches" \
          -d '{
            "ref": "main",
            "inputs": {
              "title": "Automated workflow failure issue for commit ${{ github.sha }}",
              "body": "This issue was automatically created by the GitHub Action workflow **${{ github.workflow }}**"
            }
          }'
```

While not an extensive set, these couple of examples should help you leverage GitHub functionality through your jobs. Up next, I’ll discuss another way to leverage the jobs in your workflow to do more without having to create other workflows or actions. The following section discusses a higher-level way to automatically create multiple jobs based on combinations of different values, using GitHub Actions’ *matrix strategy*.

# Using a Matrix Strategy to Automatically Create Jobs

Early in this book, we talked about the different kinds of triggers that could cause a workflow (and in turn jobs) to execute. Common examples include GitHub events such as pushes, pull requests, etc. But there are also options such as scheduling via cron and dispatching workflows manually or via other workflows as covered in [Chapter 12](ch12.html#ch12).

In addition to the triggering event, you might also want to have a job automatically run for all combinations of certain values. This is where the *matrix strategy* for dynamically creating jobs is useful. Common use cases might be to run tests on your code across multiple environments, such as dev, test, and release. Another example could be running tests across multiple operating systems or across the combinations of dev for each OS, test for each OS, and prod for each OS.

[Chapter 8](ch08.html#ch08) touched lightly on the mechanism of using a matrix strategy, but it deserves a more advanced discussion on the mechanics here.

## One-Dimensional Matrices

The way to tell GitHub Actions that you want to use a matrix approach for a job in your workflow is by including the `strategy` clause with a designation of `matrix` and then defining at least one variable that points to an array of values. The next listing shows an example of a one-dimensional matrix defined to open a GitHub issue for each of two products—*prod1* and *prod2 (*lines 8–10*)*:

```yml
name: Create issues across prods

on:
  push:

jobs:
  create-new-issue:
    strategy:
      matrix:
        prod: [prod1, prod2]  # Define the products for which issues should be created

    uses: rndrepos/common/.github/workflows/create-issue.yml@v1  # Using a reusable workflow to create issues
    secrets: inherit  # Inherit secrets for this job
    with:
      title: "${{ matrix.prod }} issue"  # Dynamic title based on matrix
      body: "Update for a level"  # Fixed issue body text

  report-issue-number:
    runs-on: ubuntu-latest
    needs: create-new-issue  # Ensure this job runs after create-new-issue
    steps:
      - run: echo "Created issue number: ${{ needs.create-new-issue.outputs.issue-num }}"
```

When this workflow is executed, it will dynamically create two separate jobs for the job with the matrix strategy—one for *prod1* and one for *prod2*. [Figure 13-1](#jobs-created-across-p) shows the jobs from the Actions run.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1301.png)<br>
**Figure 13-1. Jobs created across prod dimension**

## Multi-dimensional Matrices

You can take this a step further and add another dimension. Suppose that instead of having just one issue per product, you want to have one issue generated *per level of each product*. If the levels are *dev, stage*, and *prod* (for production), then the code to create jobs for the various combinations of product and level might look like this (notice the additional variable and array for the level dimension in line 11):

```yml
name: Create issues across prods and levels

on:
  push:

jobs:
  create-new-issue:
    strategy:
      matrix:
        prod: [prod1, prod2]  # Loop through product types
        level: [dev, stage, rel]  # Loop through levels (dev, stage, release)

    uses: rndrepos/common/.github/workflows/create-issue.yml@v1  # Reusable workflow to create issues
    secrets: inherit  # Inherit secrets for this job
    with:
      title: "${{ matrix.prod }} issue"  # Dynamic title based on matrix.prod
      body: "Update for ${{ matrix.level }}"  # Dynamic body based on matrix.level

  report-issue-number:
    runs-on: ubuntu-latest
    needs: create-new-issue  # Ensure this job runs after the issue creation
    steps:
      - run: echo "Created issue number: ${{ needs.create-new-issue.outputs.issue-num }}"
```

[Figure 13-2](#jobs-created-across-t) shows the jobs generated and run from the combination of the two dimensions defined in the code. In this case, there were six unique jobs generated for the product of the prods and levels.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1302.png)<br>
**Figure 13-2. Jobs created across the prods and levels**

There are additional ways you can generate the values of the dimensions that don’t require hard-coding. For example, in the next listing, I’m iterating over values passed into a workflow by referencing a payload provided via a GitHub *context*:

```yml
name: create issues from context

on:
  repository_dispatch:
    types:
      - level_updated

jobs:
  create-issues:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        level: ${{ github.event.client_payload.levels }}

    permissions:
      issues: write

    steps:
      - name: Create issues
        run: |
          gh issue create \
            --title "Issue for ${{ matrix.level }}" \
            --body "${{ matrix.level }} updated" \
            --repo ${{ github.repository }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Notice that in lines 3–6, the workflow is set up to be triggered by a `repository_​dis⁠patch` event with a custom type of `level_updated`. Then, on line 14 the values for the matrix variable `level` come from a special value passed in through the `cli⁠ent_​payload` portion that, in turn, comes in through the GitHub event. The values are dereferenced for the title and body of the issue in lines 23 and 24, respectively.

Here’s code that can invoke this workflow. Notice that the `event_type` matches the one for the trigger in the workflow. Also, the `client_payload` is defined with a key of `levels` that has the array of the two levels as values:

```yml
curl -X POST \
-H "Authorization: Bearer ${{ secrets.PAT }}" \
-H "Accept: application/vnd.github.v3+json" \
"https:/https://learning.oreilly.com/api.github.com/repos/${{ github.repository }}/dispatches" \
-d '{"event_type":"level_updated", "client_payload":{"levels":["dev","test"]}}'
echo ${{ github.repository }}...${{ github.event.repository }}
```

When this code is executed, the values passed in for the levels will be parsed, and a job will be created for each one, as shown in [Figure 13-3](#jobs-created-from-con).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1303.png)<br>
**Figure 13-3. Jobs created from context matrix**

[Figure 13-4](#issues-created-by-con) shows the issues created by the code.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1304.png)<br>
**Figure 13-4. Issues created by context matrix code**

# Understanding Payloads

To learn more about parts of the payloads, such as `cli⁠ent_​payload`, see [Webhook Events and Payloads](https://oreil.ly/JNcGm).

There are a few additional variations on the matrix strategy for specialized use cases. I’ll cover those next.

## Including Extra Values

The matrix declaration can have an `include` keyword with its own keys and values to add discrete combinations. This provides a method to do any of the following:

* Add values that may not fit in the standard pattern of the original matrix
* Add values that you may not want to be part of all the combinations
* Include additional dimensions for a particular job

An example of including an additional element is shown in this section of a listing:

```yml
    strategy:
      matrix:
        prod: [prod1, prod2]
        level: [dev, stage, rel]
        include:
          - prod: prod3
            level: dev
            tag: alpha
```

Note the addition of the *prod3, dev, alpha* combination via the `include` clause. If this is run in the context of the automated issue creation, the resulting jobs would look like those in [Figure 13-5](#additional-job-added).

You can also use `exclude` to exclude particular combinations from being used for a job.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1305.png)<br>
**Figure 13-5. Additional job added via include clause**

## Excluding Values

The `exclude` clause, when used with the `matrix` strategy, prevents discrete combinations of the dimensions from being made into jobs. If multiple dimensions are involved, you don’t need to specify all the possible combinations to exclude. Specifying a value for one dimension will exclude all combinations with that value.

Here’s an example of a workflow `strategy` portion that shows the `exclude` clause:

```yml
    strategy:
      matrix:
        prod: [prod1, prod2]
        level: [dev, stage, rel]
        exclude:
          - prod: prod1
            level: stage
          - level: dev
```

Note the exclusion of the *prod1, stage* combination and the exclusion of the *dev* level that will exclude it for all other combinations/dimensions. If this is run in the context of the automated issue creation, the resulting jobs would look like those in [Figure 13-6](#jobs-excluded-via-the).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1306.png)<br>
**Figure 13-6. Jobs excluded via the `exclude` keyword**

Only jobs that were not impacted by the `exclude` criteria were created—no `dev` jobs and no job for `prod1,stage`. With multiple combinations of jobs spun up, it’s important to understand how failure cases can be handled.

## Handling Failure Cases

When a job created as the result of a matrix strategy fails, there are two possible responses:

* Current job fails, but remaining jobs are allowed to proceed
* Current job fails, and all remaining jobs are canceled

Jobs can specify a strategy as part of the job definition to handle failures for either case. For the case where you want the remaining jobs to proceed, you set the `continue-on-error` clause to `true`:

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    continue-on-error: true
```

For the case where you want the remaining jobs to be canceled, you can set the `fail-fast` value to `true` in the strategy section:

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
```

One last strategy option is setting the maximum number of parallel jobs that should run at once.

## Defining Max Concurrent Jobs

GitHub, by default, will maximize the number of jobs that can be executed in parallel, based on the availability of runner systems. You can override that by setting the `max-parallel` property of the strategy. The example code that follows shows how to set the most parallel jobs that can be executed simultaneously to 3:

```yml
jobs:
  my_matrix:
    strategy:
      max-parallel: 3
```

In summary, there are a number of different options you can set with a strategy configuration for a job. The ability to define matrices, and have GitHub Actions dynamically create jobs as it iterates over the combinations of the dimensions of the matrix, provides a powerful construct for executing a large number of jobs easily.

For the last part of this chapter, I’ll show you how to leverage a key approach to have your jobs and steps run in customizable environments—using containers. And, as a bonus, you’ll see how to spin up a container as a service for your workflow to leverage.

# Using Containers in Your Workflow

Most of the time, you may default to just running your workflows on the VMs provided by GitHub or on your own self-hosted runners. However, containers are another option for this. In fact, GitHub Actions allows for multiple ways of using containers in your workflow. This gives you increased flexibility since you can have images with different environments, tooling, etc., used for running jobs and steps. Instead of being restricted to the runner’s environment or having to extend it with more configuration, you can run your workflow code in the customized container’s environment.

The most basic way to run a container in your workflow is via a step that uses the run command to execute Docker, for example:

```yml
jobs:
  info:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: build with gradle
      run: sudo docker run --rm -v "$PWD":/workdir
           -w /workdir centos:latest ls -laR
```

Or you can use a specific predefined action to run Docker commands, such as [the Docker Run action](https://oreil.ly/OoFde).

Beyond these explicit approaches, there are three *built-in* ways to leverage a container in your workflow: as the environment for a *job’s steps* to run in, as the environment for an *individual step* to run in, and/or to spin up a *service for use by jobs* in your workflow.

# Use a Linux Runner

If your workflow uses Docker container actions, job containers, or service containers, then you must use a Linux runner. For GitHub-hosted runners, that means using an Ubuntu runner. For self-hosted runners, that means using a Linux system with Docker installed.

## Using a Container as the Environment for a Job

At the job level, you can define a container to be installed on the runner. Then, instead of executing directly on the runner, any steps in the workflow will be run in the installed container. The advantage for the job is that you can precisely define the environment, applications, and versions via the container. You don’t have to do additional custom configuration on the runner or rely on the available defaults.

For example, currently, if I do a Go build on the default `ubuntu-latest` runner without any other setup, the Go version is 1.20.3. Suppose that I want to build with a Go container with all the tools in it based around Go 1.20. I could use a container for the job, as shown in the next listing:

```yml
jobs:
  info:
    runs-on: ubuntu-latest
    container: golang:1.20.0-alpine
    steps:
    - uses: actions/checkout@v4
    - name: get info
      run: "go version"
```

Notice that for the job, you still need to specify the `runs-on` identifier. The reason is that the container has to have a system to execute on that can run Docker. Per the earlier note, if you are using GitHub-hosted runners, you have to use an Ubuntu runner. And if you’re using self-hosted runners, you have to use a Linux system with Docker on it.

The `container` line is what tells the job which image to base the container on. Here it’s specifying the particular *golang* image I want. This single-line format is the simplest form of the container specification. If you need to add credentials, environment variables, volume mounts, etc., you’ll need a longer form. Here’s an example using node:

```yml
jobs:
  node-prod:
    runs-on: ubuntu-latest
    container:
      image: node:20-alpine:3.16
      env:
        NODE_ENV: production
      ports:
        - 80
      volumes:
        - source_data:/workdir
      options: --cpus 2
```

[Table 13-2](#options-for-using-con) outlines what the different options are and how they are used.

Table 13-2. Options for using containers at the job level

| Option | Meaning | Usage |
| --- | --- | --- |
| image | Image to base the container on. | image: *image-path* |
| credentials | Map of username and password/token if registry requires authentication. Same values that would be used with *docker login.* | credentials:  username: *user*  password: *password/token* |
| env | Map of environment variables for the container. | env:  *NAME: value* |
| ports | Array of ports to expose on container. | ports:    - *local:container* |
| volumes | *Array of volumes for the container to use; can be named volumes, anonymous volumes, or bind mounts from host.* | volumes:  source:destinationPath |
| options | Additional standard [Docker container options](https://oreil.ly/X5Col) (but *--network* and *--entrypoint* are not supported). | options:  *--option-name value* |

The `credentials` option deserves a bit more explanation. If you need authentication to pull the image from a registry, you will need to supply the username value and a value for a password or token (to access the registry). To be most secure, you should generate a token through whatever means the registry provides. And to ensure these are managed in a secure way for your workflow, you should create secrets in the repository to store the values. Or, if the values don’t need to be secure, you can create a repository variable for them.

The listing that follows shows an example of using the `credentials` option to access a private image hosted on *quay.io*:

```yml
jobs:
  lint-tool:

    runs-on: ubuntu-latest
    container:
      image: quay.io/techupskills2/xmltools:1.0.0
      credentials:
        username: ${{ secrets.QUAYIO_ROBOT_USER }}
        password: ${{ secrets.QUAYIO_ROBOT_TOKEN }}

    steps:
      - uses: actions/checkout@v4
      - name: run xmllint
        run: xmllint web.xml
```

This example also shows another use case for using a container—pulling in custom tools. In this case, the workflow is leveraging a custom image that has XML tools in it to run a linter against an XML file in the current repository.

# Default Shell

Note that the default shell for `run` steps within a container is *sh*, not *bash*. You can use the `defaults` property for a job to change this.

You can also do a similar use case with a container at the level of an individual step. Your options there are more limited, but it can provide finer-grained control. If you have a container defined for a step, as well as a container defined for a job, the container defined as part of the step will take precedence when the step is executing. See the next section for details about how to run a container with a step.

## Using a Container with a Step

At a lower level, you can also use a container with a step. The format for this starts with a `uses: docker://<full path of image>` line. It also allows for the option to provide a specific `entrypoint` with arguments that will override an existing one in the container (if one is present). This is done via the `with` key. The next listing shows an example of using a container, for the XML tooling from the previous listing, with a step:

```yml
jobs:
  lint-tool:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: run xmllint
        uses: docker://bclaster/xmltools:1.0.0
        with:
          entrypoint: xmllint
          args: web.xml
```

Beyond these execution-focused approaches, there is one other useful way that GitHub Actions provides for leveraging containers in workflows—automatically spinning them up as fully accessible services.

## Running Containers as Services in a Job

Containers can be used to host services for jobs in a workflow. This is useful for working with applications such as databases, web services, and memory caches. And if you have multiple such applications needed by the job, you can have multiple containers, configured as different services in the same job.

There are several advantages for your workflow when using containers this way:

* The runner system will automatically manage the lifecycle of the service containers.

  + GitHub creates a new container for each service configured in the workflow.
  + GitHub destroys the service container when the job completes.

* It will also automatically create a Docker network for the service containers.

* If the job runs in a container, or your step uses container actions:

  + Docker automatically exposes all ports between containers on the same Docker bridge network.
  + The hostname is automatically mapped to the label configured in the workflow.
  + The service container can be referenced by the hostname.

* If the job is configured to run directly on the runner and the step doesn’t use a container action:

  + You have to map any required service container ports to the Docker host (the runner).
  + The service container can be accessed using localhost and the mapped port.

* Steps in a job can communicate with all service containers that are part of the same job.

Here’s an example of defining a MySQL container as a service for a job. The steps in the job can then access the service to do database actions (such as preparing to run integration tests in this case):

```yml
jobs:
  integration-tests:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: ${{ secrets.MYSQL_ADMIN_PASS }}
          MYSQL_DATABASE: inttests
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping"
                 --health-interval=10s
                 --health-timeout=5s
                 --health-retries=3

    steps:
      - uses: actions/checkout@v4
      - name: Verify integration tests db exists
        run: mysql --host 127.0.0.1 --port 3306
         -u${{ vars.MYSQL_ADMIN_USER }}
         -p${{ secrets.MYSQL_ADMIN_PASS }}
         -e "SHOW DATABASES LIKE 'inttests'"
```

Finally, note that for a more advanced option, you can create a Docker container action. Docker container actions are described in more detail in [Chapter 11](ch11.html#ch11) on creating custom actions.

# Conclusion

In this chapter we’ve covered some additional techniques you can employ to expand the scope of your workflow’s execution beyond the basic patterns.

The ability to reference GitHub components and drive operations in GitHub via the CLI, a script, or an API call greatly expands the set of functionality you can leverage directly within the GitHub environment. It also simplifies performing tasks in GitHub that don’t warrant calling an action or creating a separate workflow to handle.

The extended discussion on matrix strategy showed how to leverage that capability to its fullest to automatically generate jobs to cover a wide mix of different dimensions. It also covered how to deal with situations where you need to include/exclude items from the matrix combinations and decide how to handle failure.

The last part of the chapter explored the various ways you can take advantage of containers in your workflows, jobs, and steps. It also included an example of how to run a container as a service. This can significantly simplify situations where you need a simple service for your workflow to access.

While the need to use these techniques may not be encountered often, it is almost certain that you will encounter a need for one or more if you continue your journey with GitHub Actions. So you can file this chapter away for future reference for that need.

In the last chapter of the book, I’ll provide some guidance for another specialized use case—migrating from existing CI/CD workflows in other providers to GitHub Actions.

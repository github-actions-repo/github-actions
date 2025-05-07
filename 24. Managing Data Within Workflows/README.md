# Chapter 7. Managing Data Within Workflows

It is rare today that a complete set of work is accomplished with a single job or project. Think about a typical CI/CD pipeline. You will usually have a job that does the building, a job for packaging, multiple jobs for testing, and so on. But even though these are individual jobs, they still need to be able to pass data and files between them. For example, the build job produces a module from source code that then needs to be tested and combined with other modules into a deliverable for the customer. Or jobs in a workflow may use outputs from a setup job as inputs or dependencies for configuration.

To accomplish this transfer of data and content, the separate jobs must have access to the intermediate results along the way. The jobs must be able to get to the various inputs, outputs, and files throughout the run of the larger process.

GitHub Actions provides syntax for capturing, sharing, and accessing inputs and outputs between jobs and steps in workflows. Additionally, it provides functionality for managing intermediate files or modules, which it calls *artifacts*. Actions provides the ability to persist artifacts created during a workflow run. Jobs within the same workflow can then access the artifacts and use them, like the projects in a pipeline.

Actions also provides the ability to cache collections of content to speed up future runs. This can be provided via explicitly calling the *cache* action or, in many cases, using a *setup* action (such as *setup-java*) that has caching functionality built in.

This chapter will guide you through the detaisettingls of managing inputs, outputs, artifacts, and caches in your workflows with the following sections:

* Working with inputs and outputs in workflows
* Defining artifacts
* Uploading and downloading artifacts
* Using caches in GitHub actions

First up is learning about how you can navigate inputs and outputs.

# Working with Inputs and Outputs in Workflows

Within a workflow, you may need to access inputs from the workflow itself, or inputs and outputs to be shared between steps or between jobs. To do this, there is syntax you should be familiar with. You need this to capture, access, and de-reference the values appropriately. In the following sections, I’ll show you how to define and reference inputs, capture and share outputs from a step, capture and share outputs from a job, and capture output defined for an action called in a step.

## Defining and Referencing Workflow Inputs

The term *inputs* here refers to explicit values supplied by a user or process to the workflow. It doesn’t mean the values you get from contexts or from default environment variables.

When inputs have been explicitly defined, they can be referenced with the syntax *${{ inputs.<input-name> }}*. The following listing shows an example of a job accessing inputs provided for two different kinds of triggers—*workflow_call* and *workflow_​dis⁠patch*:

```yml
on:
  # Allows you to run this workflow from another workflow
  workflow_call:
    inputs:
      title:
        required: true
        type: string
      body:
        required: true
        type: string

  # Allows you to call this manually from the Actions tab
  workflow_dispatch:
    inputs:
      title:
        description: 'Issue title'
        required: true
      body:
        description: 'Issue body'
        required: true

jobs:
  create_issue_on_failure:
    runs-on: ubuntu-latest

    permissions:
      issues: write
    steps:
      - name: Create issue using REST API
        run: |
          curl --request POST \
          --url https:/https://learning.oreilly.com/api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          --header 'content-type: application/json' \
          --data '{
            "title": "Failure: ${{ inputs.title }}",
            "body": "Details: ${{ inputs.body }}"
            }' \
          --fail
```

The *workflow_call* trigger allows this workflow to be called from another workflow, making it a *reusable workflow*. (Reusable workflows are discussed in [Chapter 12](ch12.html#ch12).) The *workflow_dispatch* trigger creates a way to invoke (or dispatch) the workflow directly from the Actions interface in your repository. (More info on advanced triggers including these can be found in [Chapter 8](ch08.html#ch08).)

Regardless of which type of trigger causes the workflow to execute, you can access the input values in the same way in the body of the job—via *inputs.<value-name>*. (Here *inputs* is one of the *contexts* provided by GitHub Actions, as explained in [Chapter 6](ch06.html#ch06).)

# Untrusted Input

Always be careful when dealing with input values that might be able to be compromised, such as with script injections. Even some context-supplied values that are defined by users can be suspect. [Chapter 9](ch09.html#ch09) on security provides more information about this and techniques to guard against problems.

## Capturing Output from a Step

You can capture output from a step by defining the output as an environment variable and writing it to *GITHUB_OUTPUT*. Before you do this, though, you need to make sure to add the *id:* to the step with a value if it’s not already there. The value for the id field becomes part of the path to reference the environment variable’s value in other steps.

The following job code has two steps. The first one named *Set debug* has been assigned an id of *set-debug-stage*. In its last line, it sets the environment variable *BUILD_STAGE* and then dumps that into the special file that GitHub Actions maintains on the runner designated by *$GITHUB_OUTPUT*:

```yml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - name: Set debug
        id: set-debug-stage
        run: echo "BUILD_STAGE=debug" >> $GITHUB_OUTPUT

      - name: Get stage
        run: echo "The build stage is
${{ steps.set-debug-stage.outputs.BUILD_STAGE }}"
```

The second step gets the value from the other step by referencing the hierarchy path of *steps.<step id>.outputs.<env var name>*.

# Don’t Use set-output

Previously, GitHub Actions supported special workflow commands to capture output. This was done via the *set-output* command. For example:

```yml
- name: Set output
  run: echo "::set-output name={name}::{value}"
```

That method was deemed unsafe to continue using. So any such code should be changed to use the new environment files:

```yml
- name: Set output
  run: echo "{name}={value}" >> $GITHUB_OUTPUT
```

## Capturing Output from a Job

Capturing output from a job can build on output from a step. Suppose that I want to change the previous example so that the second *Get stage* step is in a separate job. To pass the information back from the first job, I need to define a new `outputs` section for that job with the output value pulled from the step. The `outputs` section consists of a *key:value* pair. The key is the reference for other jobs to get to the output. The value is the workflow path to get to the output from the step.

Using this approach on the previous example, the *setup* job would be updated with an `outputs` section to capture the output from the step and persist it:

```yml
jobs:
  setup:
    runs-on: ubuntu-latest

    outputs:
      build-stage: ${{ steps.set-debug-stage.outputs.BUILD_STAGE }}

    steps:
      - name: Set debug
        id: set-debug-stage
        run: echo "BUILD_STAGE=debug" >> $GITHUB_OUTPUT
```

I could then add a new job to get that output and report it back:

```yml
 report:
    runs-on: ubuntu-latest
    needs: setup
    steps:
      - name: Get stage
        run: echo "The build stage is
 ${{ needs.setup.outputs.build-stage }}"
```

To get the value from the first job, the step in the second job uses the `needs.<job>.outputs.<output value name>` syntax. This leverages the *needs* context, which captures outputs from other jobs that have a dependency relationship. (Contexts are discussed more in [Chapter 6](ch06.html#ch06).) Note that the sequencing is established by the `needs` setup statement in the second job. It’s important to ensure the first job has completed before we try to access its output.

# A Cleaner Way to Deal with Outputs

Since the syntax to access an output can be extensive, it may be simpler to reference it via an environment variable in a job. An example of updating the previous job for this is shown here:

```yml
report:
  runs-on: ubuntu-latest
  needs: setup
  steps:
    - name: Get stage
      env:
        BUILD_STAGE: ${{ needs.setup.outputs.build-stage }}
      run: echo "The build stage is $BUILD_STAGE"
```

## Capturing Output from an Action Used in a Step

When a step uses an action and that action has outputs defined (via its *action.yml* metadata), you can capture that output in the same way. For example, I have a workflow that uses a [changelog action](https://oreil.ly/BFrQr) to help generate a nicely formatted changelog via conventional commits. The step looks like this in the code:

```yml
    - name: Conventional Changelog Action
      id: changelog
      uses: TriPSs/conventional-changelog-action@v3.14.0
```

As shown in the following excerpt from the action’s *[action.yml](https://oreil.ly/r2e1y)* file, an output is defined named `version`:

```yml
outputs:
  changelog:
    description: "The generated changelog for the new version"
  clean_changelog:
    description: "The generated changelog for the new version
 without the version name in it"
  version:
    description: "The new version"
```

Given this setup, I can capture the `version` output (from the step in my workflow with `id: changelog`) to pass back from the job with a similar approach as before:

```yml
jobs:
  build:
    runs-on: ubuntu-latest

    # Map a step output to a job output
    outputs:
      artifact-tag: ${{ steps.changelog.outputs.version }}
```

Being able to capture and share input and output values throughout your workflow provides a way to transfer simple data values between steps and jobs. It is one way of enabling the jobs in your workflow to work together. A larger part of ensuring that jobs in a workflow can function together to process content (and implement flows such as pipelines) is being able to persist and access objects created during job runs. Normally, when a job is done and a runner goes away, you would lose any files created as a result of running the job. But GitHub Actions provides functionality to allow you to persist and access created content after the job or workflow is done. This is done through creating and accessing artifacts.

# Defining Artifacts

*Artifacts*, as Actions defines them, are simply files or collections of files, created as the result of a job or workflow run and persisted in GitHub. The persistence is usually so that the artifact can be shared with other jobs in the same workflow, although you might also persist an artifact to have access to it via the UI or a REST API call after the run was complete.

Examples of artifacts could include modules produced in a build job that then need to be packaged or tested by other jobs. Or you might have log files or test output that you want to persist to look at outside of GitHub. While these items can be persisted, they cannot be persisted indefinitely. By default, GitHub will keep your artifacts (and build logs) around for 90 days for a public repository.

# Configuring the Artifact Retention Policy

If you have permissions on the repository, the 90-day period is configurable. For a public repository, 90 days is the max, but the duration can be set to anywhere between 1 and 90 days. For a private repository, the duration can be set between 1 and 400 days. Note that this only applies to new artifacts and log files, not existing ones. Organizations and enterprises can set limits at their levels that you may not be able to override.

As mentioned in [Chapter 1](ch01.html#ch01), GitHub provides you with a certain amount of storage at no cost for artifacts depending on your particular plan. If you go over that amount, you are charged. Unlike usage minutes for actions, storage costs are cumulative over the time you have the artifacts stored in GitHub.

# GitHub Packages

GitHub has another product that should not be confused with artifacts—GitHub Packages. GitHub Packages is a repository for multiple different kinds of packages, including packages for the following:

* containers
* RubyGems
* npm
* Maven
* Gradle
* NuGet

Also, with packages, GitHub charges you for data transfer. With artifacts, it does not.

To persist an artifact created by a job, there are additional steps you must add to your workflow. The next section explains how that works.

# Uploading and Downloading Artifacts

Persisting artifacts is necessary when you want to access an artifact between the jobs in your workflow. When each job runs, you must specify the `runs-on` clause to tell GitHub where to execute the code for the job. This means that if you’re using a separate runner instance for each job, the environment is spun up, used, and then removed, along with any artifacts created in the process.

For example, suppose we have the workflow code shown in the listing that does a simple build of a Java source program with Gradle.

```yml
name: Simple Pipe

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 1.8
      uses: actions/setup-java@v4
      with:
        java-version: 1.8
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew build
```

The format and structure of the workflow should look familiar. There is the `on` clause describing what events trigger the workflow, followed by the `jobs` section. There is a single `build` job that uses the *checkout* action to get the code from the repository then sets up a Java Development Kit and does a Gradle build.

To make items created during the run of a job available after the run is complete, you need to add additional code to your workflow. Not surprisingly, GitHub provides an action for that. It’s called *upload-artifact* and can be found in the [GitHub actions area](https://oreil.ly/wzBiq). (See [Figure 7-1](#action-to-upload-a-bu).)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0701.png)<br>
**Figure 7-1. Action to upload a build artifact**

Notice the large rectangular button in the upper right labeled *Use latest version*. Selecting this is an easy way to get code required to start using the action in your workflow. When you select this, you get a dialog with the basic `uses` statement and a suggested `name` statement that you can simply copy and paste directly into your workflow and save some typing. (This code will represent the simplest use case, though.) If you select the small down arrow at the right end of that button, you can select the same kind of example code for previous versions of the action. When you do this, a yellow banner at the top of the page will remind you that you’re viewing an older version of the action ([Figure 7-2](#getting-sample-code-f)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0702.png)<br>
**Figure 7-2. Getting sample code for basic usage of a previous version**

Pasting or typing in the code into your workflow can be done with the standard edit mechanisms, including the edit interface in the browser. You will need to make sure that when you paste, you get the alignment (number of spaces for indenting) right. The GitHub interface will flag the code with the usual red wavy line if it isn’t aligned correctly ([Figure 7-3](#misaligned-text-in-br)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0703.png)<br>
**Figure 7-3. Misaligned text in browser editor**

While the code has been added to invoke the action and upload the build artifact, we’re not done yet. We still need to add parameters and values to tell the workflow which artifact to upload. That’s covered in the next section.

## Adding Parameters

Some actions, such as *checkout*, do not require additional parameters (although they may take optional ones). The checkout action assumes that you want to check out the code from the repository where the workflow is running.

However, many actions require one or more parameters to be useful. In most cases, you can find example usage information for an action on the action’s home page. [Figure 7-4](#usage-info-for-upload) shows that file for the *Upload a Build Artifact* action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0704.png)<br>
**Figure 7-4. Usage info for upload-artifact action**

Depending on the action, there may also be other informative information on this page, such as *What’s new*, some sort of listing of known issues, and licensing info.

The definitive source for usage information, though, is the actual *action.yml* file that is part of the code repository for the action. This file is the specification for how to interact with the action. It contains the complete set of options and related information, such as default values. This file is easy to read and can always be found in the root of the repository ([Figure 7-5](#actionyaml-file-for2)). Sometimes, an action author may also include the entire file, or a link to it, on the marketplace page.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0705.png)<br>
**Figure 7-5. action.yml file for upload-artifact**

I’ll describe this file and its structure in more detail in [Chapter 11, “Creating Custom actions”](ch11.html#ch11).

From this file, you can see that the *Upload a Build Artifact* action takes the parameters shown in [Table 7-1](#parameters-for-the-ac).

Table 7-1. Parameters for the action to upload a build artifact

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| name | yes | artifact | Name of the artifact. |
| path | yes | none | File system path to what you want to upload. |
| if-no-files-found | no | warn | What to do if there are no files in the path you specified: a value of `error` means stop with an error; a value of `warn` means report the issue but don’t fail; a value of `ignore` means don’t fail and don’t print a warning—just keep going. |
| retention-days | no | 0 (which means use the repository default; see description) | Number of days before the artifact will expire (be removed from GitHub). Can be between 1 and 90 to indicate a specific number of days or 0 to use the default. |

# Setting the Default Number of Days for Retaining Artifacts and Logs

As previously mentioned, there are defaults for the number of days that GitHub allows for retaining artifacts and logs of runs.

To change this value, you would go to the Settings tab in the Repository, find the Actions menu selection on the lefthand side and click it. There are two submenus that will then become visible: General and Runners. Select General.

You’ll then see a section titled *Artifact and log retention*. In the input field, you can enter a number of days that you want for the default—from 1 to 90 for this public repository. And then click the Save button. [Figure 7-6](#artifact-retention-se) shows this section on an example repository’s page.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0706.png)<br>
**Figure 7-6. Artifact retention settings**

Getting back to the process of uploading an artifact, I can add code like the example usage into our workflow. The next listing shows code for a job that builds an artifact (in this case, a Java JAR file) and uploads it to the storage area:

```yml
20. jobs:
21.   build:
22.
23.     runs-on: ubuntu-latest
24.
25.     steps:
26.     - uses: actions/checkout@v4
27.     - name: Set up JDK 1.8
28.       uses: actions/setup-java@v4
29.       with:
30.         java-version: 1.8
31.     - name: Grant execute permission for gradlew
32.       run: chmod +x gradlew
33.     - name: Build with Gradle
34.       run: ./gradlew build
35.     - name: Upload Artifact
36.       uses: actions/upload-artifact@v4
37.       with:
38.         name: greetings-jar
39.         path: build/libs
```

The actual build starts with the step at line 33. By default, the output from the Gradle build will go to the *build/libs* directory in the build area on the runner.

Lines 35–39 define the step that uses the `upload-artifact` action. The `with` clause starting on line 37 defines two arguments to pass to the action. The first one starting with `name` is the identifier to use for the overall artifact, even if it includes multiple files. The second one, `path`, is the location to collect files from on the runner system.

After the workflow is run, the artifact produced during the build is available at the bottom of the page for the run of the workflow ([Figure 7-7](#artifact-produced-fro)).

The artifact has been uploaded to GitHub (via the *upload-artifact* step) and is now available to download manually, delete, or use with another job in the same workflow. Downloading it manually is as simple as clicking the artifact name. Deleting it is as simple as clicking the trash can icon on the right side and confirming that you do want to delete it.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0707.png)<br>
**Figure 7-7. Artifact produced from build**

In many cases, though, you may want to use the artifact in another job. Consider, for example, a CI/CD pipeline process defined in a workflow that builds the artifact and then runs it as part of a test. To do the testing, you can add a second job *test-run* to the workflow.

First, define a new job:

```yml
test-run:
  runs-on: ubuntu-latest
```

To test the artifact, you need to ensure that the artifact is built first. The build is handled by the existing build job. So you need to be sure that the job has completed first. Recall that in GitHub Actions, jobs run in parallel by default. To make sure that the build has run and completed, you’ll add the `needs` clause, as shown next:

```yml
test-run:
  runs-on: ubuntu-latest
  needs: build
```

At this point, the workflow needs to be able to access the artifact that was previously uploaded. You might think that since you can see it and access it via clicking it in the output of the run, it is also immediately available to your job and its steps. However, just as the artifact had to be uploaded after the initial build step to persist it, it now has to be downloaded by any job that wants to use it.

The reason for this is that, as noted previously, each job runs in a separate VM, so each job’s environment is separate from the others. Fortunately, just as there was an *upload-artifact* action available, there is also a *download-artifact* action. [Figure 7-8](#action-for-downloadin) shows the screen for that action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0708.png)<br>
**Figure 7-8. Action for downloading a build artifact**

The corresponding *action.yml* file (shown in [Figure 7-9](#actionyaml-for-descr)) is pretty simple for this one—basically a name of the artifact to download and an optional path to download it to. If a path is not specified, the artifact will be downloaded to the current directory.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0709.png)<br>
**Figure 7-9. action.yml for describing what’s needed to download an artifact**

Adding this into the workflow job for the artifact that was previously created yields this code:

```yml
test-run:
  runs-on: ubuntu-latest
  needs: build
  steps:
  - name: Download candidate artifacts
    uses: actions/download-artifact@v4
    with:
      name: greetings-jar
```

As before, there’s an optional `name` for the step, followed by a `uses` clause to select the action via its path (relative to github.com) with tag *v2*. Next, there’s the `with` clause with the name of the input parameter from the *action.yml* and its value.

Now, all that is left for this simple job is to run a step to actually test the artifact. For illustration purposes, I do this with an additional step in the job that simply runs the application (java in this case) as a shell command:

```yml
test-run:
  runs-on: ubuntu-latest
  needs: build
  steps:
  - name: Download candidate artifacts
    uses: actions/download-artifact@v4
    with:
      name: greetings-jar
  - shell: bash
    run: |
      java -jar greetings-actions.jar
```

The upload/download actions work well for smaller sets of *output* artifacts that need to be persisted. But for many activities, such as building, your workflow jobs or the actions they use may need a larger set of dependencies downloaded on the runner. To help with managing and speeding up this process, there is one more strategy we can use that’s beneficial—caching.

# Using Caches in GitHub Actions

GitHub Actions includes the ability to cache dependencies to make actions, jobs, and workflows more efficient and faster to execute. Common packaging and dependency management tooling such as npm, Yarn, Maven, and Gradle create a cache of their downloaded dependencies (usually under hidden areas such as *.m2* for Maven and *.gradle* for Gradle). The caching that is built into these applications saves time when using the same tooling on the same machine since the dependencies don’t have to be downloaded again.

However, if you’re using runner systems hosted on GitHub, each job in an action starts in a clean execution environment, downloading dependencies again by default. This results in using more network bandwidth and longer runtimes. For paid plans, this can ultimately increase the cost of using actions. To help with these issues, GitHub Actions can cache dependencies used frequently by these applications.

# Warnings Regarding Using Caching in GitHub Actions

The following is from the [GitHub Actions documentation](https://oreil.ly/9jShn) on using caching:

* We recommend that you don’t store any sensitive information in the cache. For example, sensitive information can include access tokens or login credentials stored in a file in the cache path. Also, command line interface (CLI) programs like `docker login` can save access credentials in a configuration file. Anyone with read access can create a pull request on a repository and access the contents of a cache. Forks of a repository can also create pull requests on the base branch and access caches on the base branch.
* When using self-hosted runners, caches from workflow runs are stored on GitHub-owned cloud storage. A customer-owned storage solution is only available with GitHub Enterprise Server.

There are two options for enabling the caching functionality with actions. The first option is to use the explicit [cache action](https://oreil.ly/bCJzk). The second is activating it within the various *setup-** actions, such as the [setup-java action](https://oreil.ly/50MnS) shown previously.

The first option requires more configuration but gives you explicit control over the caching. As well, it allows use across a wider set of applications. The second option requires minimal configuration and manages the creation and restoration of the cache automatically—if you are using one of the setup-* actions.

## Using the Explicit Cache Action

The GitHub [cache action](https://oreil.ly/P2ys3) is applicable to a large number of programming languages and frameworks. Most of the inputs and outputs for it are fairly straightforward (see [Table 7-2](#cache-action-inputs-a)) and are also described in the code’s [*action.yml* file](https://oreil.ly/lmNZx).

Table 7-2. Cache action inputs and outputs

| Function | Name | Description |
| --- | --- | --- |
| input | `path` | A list of files/directories/patterns that specify which file system objects to include in the cache and restore from it |
| input | `key` | An explicit key to use for saving and restoring the cache |
| input | `restore-keys` | An ordered list of keys to check to indicate that a match occurred with the key; a match here is referred to as a *cache hit* |
| input | `upload-chunk-size` | Chunk size used to split up large files during upload, in bytes |
| input | `enableCrossOs​Arch⁠ive` | Optional boolean—allows Windows runners to save or restore caches that can be restored or saved respectively on other platforms |
| input | `fail-on-cache-miss` | Fail workflow if cache entry is not found |
| input | `lookup-only` | See if a cache entry exists for the given input(s) (key, restore-keys) without downloading the cache |
| output | `cache-hit` | Simple boolean to indicate if an exact match is found for a key |

To see how using the cache action works in practice, I’ll provide a simple example for caching and restoring Go dependencies, starting with how to create a cache.

### Creating caches

Here’s example code for a step in a simple Go build workflow that invokes the cache action:

```yml
- uses: actions/cache@v4
  env:
    cache-name: go-cache
  with:
    path: |
      ~/.cache/go-build
      ~/go/pkg/mod
    key: ${{ runner.os }}-build-${{ env.cache-name }}-
${{ hashFiles('**/go.sum') }}
    restore-keys: |
      ${{ runner.os }}-build-${{ env.cache-name }}-
      ${{ runner.os }}-build-
      ${{ runner.os }}-
```

The `uses: actions/cache@v4` step invokes the cache action. The input parameters to the action are passed via the `with` clause, and the input paths identify local directories on the runner to include in the cache. Those directories would vary depending on the application being used. For example, if this were a cache based on Maven, then the paths would reference *.m2* instead.

The syntax used to create the `key` needs some additional explanation. Cache keys can be made up of any combination of variables, static strings, functions, or context values up to a maximum length of 512 characters. (Contexts are discussed in [Chapter 6](ch06.html#ch06).)

The idea here is that the set of variables and computed values strung together will make up a unique key. You can include any values of your choosing. Looking at the `key` line in the previous listing, it can be understood as follows:

> *`runner.os`*
>> Provided by the GitHub Actions runner context, this is the type of the host OS on the runner. If running on a Linux environment, then the value would be *Linux,* for example.
>
> *`build`*
>> An indicator of the type of operation being done.
>
> *`env.cache-name`*
>> An environment variable to set the main part of the cache name.
>
> *`hashFiles`*
>> A unique value created from running a hash algorithm over the specified paths.

# runner.os in Key

You may wonder why `runner.os` doesn’t resolve to the `ubuntu-​lat⁠est` entry that is specified for the job to run on. `runner.os` is the *flavor* of the operating system (Windows, Mac, Linux) without respect to version. To include the actual specific operating system version, you would use `matrix.os` instead.

For the `hashFiles` results, debug mode shows what actually occurs (debugging mode and debugging in general are covered in [Chapter 10](ch10.html#ch10)):

```log
##[debug]::debug::Search path
'/home/runner/work/simple-go-build/simple-go-build'
##[debug]/home/runner/work/simple-go-build/simple-go-build/go.sum
##[debug]Found 1 files to hash.
##[debug]Hash result:
'b60843ce1ce1b0dc55b9e8b2d16c6dffeec7e359791af9b9cf847ee3ee50289e'
##[debug]undefined
##[debug]STDOUT/STDERR stream read finished.
##[debug]STDOUT/STDERR stream read finished.
##[debug]Finished process 1754 with exit code 0, and elapsed time
00:00:00.0833289.
##[debug]..=>
'b60843ce1ce1b0dc55b9e8b2d16c6dffeec7e359791af9b9cf847ee3ee50289e'
##[debug]=>
'Linux-build-go-cache-b60843ce1ce1b0dc55b9e8b2d16c6dffeec7e359791af9b9cf847ee3ee
50289e'
##[debug]Result:
'Linux-build-go-cache-b60843ce1ce1b0dc55b9e8b2d16c6dffeec7e359791af9b9cf847ee3ee
50289e'
```

The last two lines in the preceding listing show the generated key. While it is not required to use the `hashFiles` piece, that is commonly used to provide a unique cache for each set of content. Another approach could be simply running `hashFiles` on a list of dependencies such as in a *requirements.txt* file if you’re working in Python. As long as the file is different between instances, the generated hash would be different, and thus the cache key would be unique.

You can also use a hard-coded key that you create or derive via some other command. The following example is taken from the cache action [documentation](https://oreil.ly/bY6ot) on GitHub:

```yml
  - name: Get Date
    id: get-date
    run: |
      echo "date=$(/bin/date -u "+%Y%m%d")" >> $GITHUB_OUTPUT
    shell: bash

  - uses: actions/cache@v4
    with:
      path: path/to/dependencies
      key: ${{ runner.os }}-${{ steps.get-date.outputs.date }}
-${{ hashFiles('**/lockfiles') }}
```

In this example, the cache key is created by invoking a separate step in the job. The step before that one calls the date command and captures the result of running the command as an output value.

### Matching keys

The `restore-keys` list is optional, as the workflow will look for an exact match to the key first. If there is an exact match, then the action will restore the files from the cache into the location(s) specified in the `path` parameter (under the `with` section of the cache action invocation). When this happens, it’s called a *cache hit*. With the cache action, this is a condition you can test for if desired. Here’s an example (citing a Gradle cache step):

```yml
    - if: ${{ steps.cache-gradle.outputs.cache-hit == 'true' }}
      name: Check for cache hit
      run: |
        echo "Got cache hit on key"
```

But in case it doesn’t find an exact match with the key (referred to as a *cache miss*), you can broaden the search for a cache to use through the ordered list in the `restore-keys` section.

If a cache miss occurs and there is a `restore-keys` list, the cache action will proceed down that list (from top to bottom) looking for partial matches to the key. If it finds an exact match with a restore key, it does the same as for an exact match to the regular key—it restores the files from the cache into the location(s) specified in the path. If there is not an exact match, it searches for a partial match. If that is found, the most recent cache that matches that partial key is restored to the path locations.

So consider that we have this set as our key and restore keys in branch *dev*:

```yml
key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles(
'**/go.sum') }}
restore-keys: |
  ${{ runner.os }}-build-${{ env.cache-name }}-
  ${{ runner.os }}-build-
  ${{ runner.os }}-
```

Then the search order would be from the most specific (the `key` value) to the least specific (the `${{ runner.os }}-` pattern). [Figure 7-10](#cache-restored-from-a) shows an example of a cache restore from a key match.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0710.png)<br>
**Figure 7-10. Cache restored from a key match**

If the action recognizes that there wasn’t a match for the original key, but there was a cache that worked and allowed the job to be completed, it will do one additional step. It will automatically create a new cache with the contents of the path. For this to occur, all three of these conditions must be true:

* A cache-miss occurs.
* A restore key matches.
* The job completes successfully.

### Cache scope

Just as workflows are associated with specific branches in the GitHub repository, so are caches. The cache action will search for cache hits first by trying to match the key and set of restore keys against caches created in the branch that contains the workflow run. If it doesn’t find a cache hit in that branch, it will look for a hit in the parent branch and then upstream branches. This is useful when branches are created from other branches and inherit the workflow runs, or when doing operations like pull requests.

Caches can be shared in several dimensions. One is across the jobs in a workflow. Another is across the runs of a workflow. And a third is across branches in a repository, as previously discussed.

Depending on which dimension you want and how restrictive (or not) you want your cache to be to a particular workflow, you can define your keys to use more or less unique values. For example, if you wanted your cache to be unique for each commit or pull request, you might create your cache key using the SHA value from the [GitHub context](https://oreil.ly/I1JDK) available in Actions—something like this:

```yml
key: ${{ runner.os }}-docker-${{ github.sha }}
```

It might seem problematic to leverage a value that changes each time. But, consider a case where you have a CI/CD pipeline defined in a workflow. You might want a different cache of content for each run that is still accessible to all the jobs in the pipeline.

The behavior of looking for caches across parent branches and upstream branches is built in to GitHub Actions. If you are doing a pull request from *dev* to *main,* the cache searching would look first through the key and restore keys in the dev branch to try to find a matching cache. After that, it would do the same in the main branch.

### Cache lifecycle

Caches that have not been accessed in over seven days will automatically be removed by GitHub. While there isn’t a limit to the count of caches you can have, you are limited to 10 GB of storage for all caches in a repository. If you pass that 10 GB limit, GitHub will start getting rid of older caches (least recently accessed) to get you back down under the limit.

## Monitoring Caches

When you are on the main Workflows page in the Actions interface for a repository, you can see all of the caches that have been created by clicking the *Caches* section under *Management* on the left. [Figure 7-11](#looking-at-existing-c) shows an example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0711.png)<br>
**Figure 7-11. Looking at existing caches in the Actions interface**

You can use the Branch drop-down menu in the gray bar above the list to show caches related to a specific branch. And the Sort drop-down menu next to it can be used to sort caches by age, size, etc. See [Figure 7-12](#sorting-caches).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0712.png)<br>
**Figure 7-12. Sorting caches**

At the far right of each row that lists a cache is a trash can icon that you can use to manually delete that cache.

It is also possible to get some info on caches via API calls. For example, for an individual owner and repository, you could paste this into your browser or invoke via curl, etc.:

> *https:/https://learning.oreilly.com/api.github.com/repos/<owner>/<repository>/actions/cache/usage*

Here’s example output for one of my repositories:

```yml
{
  "full_name": "brentlaster/greetings-actions",
  "active_caches_size_in_bytes": 312329569,
  "active_caches_count": 2
}
```

There are corresponding APIs for getting the information at the enterprise and organization GitHub levels.

# GitHub CLI Extension for Managing Caches

If you are familiar with the [GitHub CLI](https://cli.github.com), there is a [gh-actions-cache](https://oreil.ly/j2kds) extension that you can install and use to manage caches from the command line.

## Activating a Cache with a Setup Action

Compared to the coding required to create and work with an explicit cache, using a cache that is part of an existing *setup* action is simple and implicit. These actions have built-in functionality for creating and using caching. They make use of the same cache functionality under the hood but have defaults for most values. Thus, they require less configuration in your code.

As an example, the *setup-java* action takes a `cache` key with a value of one of the available mechanisms for building java—`gradle`, `maven`, or `sbt`. A step created to set up a JDK with a cache for the Gradle files is shown here:

```yml
    - name: Set up JDK
      uses: actions/setup-java@v4
      with:
        java-version: '13'
        distribution: 'temurin'
        cache: 'gradle'
```

The cache parameter here is not required for the action. And caching is off by default. The cache key in this case gets automatically constructed with the following form:

```yml
setup-java-${{ platform }}-${{ packageManager }}-${{ fileHash }}
```

For the previous example code, the key is created as follows:

```log
##[debug]Search path
'/home/runner/work/greetings-actions/greetings-actions'
##[debug]
/home/runner/work/greetings-actions/greetings-actions/build.gradle
##[debug]
/home/runner/work/greetings-actions/greetings-actions/gradle/wrapper
/gradle-wrapper.properties
##[debug]Found 2 files to hash.
##[debug]primary key is setup-java-Linux-gradle-80b5a9caabdc7733a17e68fa87412ab4
c217c54462fd245cc82660b67ec2b105
::save-state name=cache-primary-key::setup-java-Linux-gradle-80b5a9caabdc7733a17
e68fa87412ab4c217c54462fd245cc82660b67ec2b105
##[debug]Save intra-action state cache-primary-key = setup-java-Linux-gradle-80b
5a9caabdc7733a17e68fa87412ab4c217c54462fd245cc82660b67ec2b105
```

The files that are hashed as the last part of the key depend on which type of build tool is being used. For the *setup-java* action, the files included in the hash are shown in [Table 7-3](#files-used-in-cache-h).

Table 7-3. Files used in cache hash

| Application | Hashed files |
| --- | --- |
| gradle | **/*.gradle*, **/gradle-wrapper.properties |
| maven | **/pom.xml |
| sbt | **/*.sbt,**/project/build.properties,**/project/**.{scala,sbt} |

Other setup actions are similar. In the *setup-go* action, there is only one builder application. So, the switch for activating the cache is simply `cache: true` in the `with` clause.

# Caching in Other Non-setup Actions

It’s worth noting that some other actions take a more proactive approach and provide specialized functions for caching. For example, the [gradle-build-action](https://oreil.ly/phs5j) provides the following:

* Automatic downloading and caching of Gradle distributions
* More efficient caching of the Gradle User Home content between runs
* Detailed reporting of cache usage and cache configuration options

[Figure 7-13](#custom-cache-info-pro) shows some examples of the custom caching resulting from use of this action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0713.png)<br>
**Figure 7-13. Custom cache info provided via the Gradle build action**

# Conclusion

In this chapter, we’ve discussed how to create, manage, share, and persist different types of data across steps, jobs, workflows, and workflow runs. These different types include inputs and outputs from steps and jobs. Such values passed between parts of a workflow are key for preserving and sharing basic data. Examples include status, versions, paths, and other simple results that need to be captured and supplied from one step, or job, into another. An output from a step might also be an action’s predefined output value that is invoked by the step. The most challenging aspect of working with these is arguably getting the syntax right when you need to de-reference them (via the `needs` clause). And you must add an id to a step to be able to reference data from it.

Beyond simple data values, artifacts in GitHub Actions provide a way to create and persist content between jobs in a workflow and after the workflow run has completed. These artifacts can be collections of files of any type and from any path that the action can access. Community actions can be used to upload and download your artifacts within the GitHub environment. Artifacts count towards your storage usage and have a default retention period. After that, they are automatically deleted. Artifacts are surfaced in the Actions interface associated with viewing the results of a run.

In addition to generated files persisted as artifacts, dependencies can be useful to persist from run to run. This avoids the time required to download or regenerate them each time and thus avoids unnecessary use of resources on each run. Many *setup* actions for tooling, such as java, Gradle, and others, provide an option to utilize caching for their standard dependencies. Having the caching as a built-in option makes it extremely simple to configure and leverage via the action in the workflow.

GitHub Actions also provides a dedicated [caching action](https://oreil.ly/Xnqtv) to directly create and use caches. When using this action, you specify how to construct a unique key value that is used to look for matching cached content. You can also supply a list of broader values to check for matches.

The GitHub Actions interface in the repository shows you caches created by/available to your workflows. The interface also allows you to delete them. Some functions are also available via the GitHub APIs.

With the understanding of how to manage the environments we have for our workflows as well as the data that is used within them, it’s time to move on to the final part of our building blocks section—understanding and managing the elements that affect the execution path and flow of your workflows.

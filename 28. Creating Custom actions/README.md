# Chapter 11. Creating Custom actions

The code that underlies a GitHub action can be very simple or very complex. It can range from a simple shell script to a collection of implementation code, test cases, and workflows (for testing, validation of content, and other CI/CD tasks). At some point after you have been using GitHub Actions for a while, you may want to start creating your own. This can be done to provide a customized version of another action or to create a specialized action from scratch.

# Before You Start...

Before starting down the path of creating your own actions, it can be helpful to search through venues like the [Actions Marketplace](https://oreil.ly/W3_72) to see if there is already an action that does what you want.

Actions can provide functionality by calling GitHub APIs, running standard shell steps, or implementing custom code. And they can execute either directly on runners (discussed in [Chapter 5](ch05.html#ch05)) or in a Docker container. This provides a high degree of flexibility when creating a custom action.

In this chapter, we’ll look at how to create and work with custom actions by covering the following topics:

* Describing the anatomy of an action
* Discussing types of actions
* Creating a simple composite action
* Creating a Docker container action
* Creating a JavaScript action
* Putting actions in the marketplace
* Working with the GitHub Actions Toolkit
* Discussing local actions

Let’s start by defining the piece that makes a repository available as an action—the interface file.

# Anatomy of an action

If you look at any of the actions on the Actions Marketplace, you’ll notice some common characteristics they all share.

* They are all in individual GitHub repositories.
* They each have a unique name.
* They have a version identifier.
* They have a README file.
* They have an *action.yml* or *action.yaml* file.

# Action Filename

Either *action.yml* or *action.yaml* is valid.

These items are consistently present even though the functionality and implementation of each action may be very different. The first four items in the preceding list are standard in most GitHub repositories. But the fifth is what makes a repository (and the code it contains) usable as an action. It is also the key file that defines the interface of the action so it can be used in workflows. While this has been briefly covered in other chapters, I’ll include a deeper dive here so you can understand how to ensure you create a suitable *action.yml* file for your custom action.

The *action.yml* file defines the input, outputs, and configuration for the action. The configuration information includes basic identification information (and optional branding information). It also includes details about the kind of environment the action is intended to execute in with any special settings for that environment. It has a well-defined format. An example *action.yml* file (for the [*cache* action supplied by GitHub](https://oreil.ly/99Ssg)) is shown in the next listing:

```yml
name: 'Cache'
description: 'Cache artifacts like dependencies and build outputs to improve workflow execution time'
author: 'GitHub'
inputs:
  path:
    description: 'A list of files, directories, and wildcard patterns to cache and restore'
    required: true
  key:
    description: 'An explicit key for restoring and saving the cache'
    required: true
  restore-keys:
    description: 'An ordered list of keys to use for restoring stale cache if no cache hit occurred for key. Note `cache-hit` returns false in this case.'
    required: false
  upload-chunk-size:
    description: 'The chunk size used to split up large files during upload, in bytes'
    required: false
outputs:
  cache-hit:
    description: 'A boolean value to indicate an exact match was found for the primary key'
runs:
  using: 'node16'
  main: 'dist/restore/index.js'
  post: 'dist/save/index.js'
  post-if: 'success()'
branding:
  icon: 'archive'
  color: 'gray-dark'
```

This format is a structure for describing how users will interact with the action. At the top is basic identifying information for the action: `name`*,* `description`, and `author`. The name and description are required. The author is not but is recommended.

This is followed by the `inputs` section. Inputs are optional. In some cases, the thing that the action is intended to work on may be self-evident. For example, the *checkout* action is intended to work on the set of code in the repository where it is used.

[Table 11-1](#input-parameter-fields) lists the various fields available for an input parameter.

Table 11-1.
Input parameter fields

| Item | Required | Description |
| --- | --- | --- |
| `< input_id >` | Yes | The name of the input must be unique, must start with a letter or _, and can contain only alphanumeric characters, -, or _. |
| `description` | Yes | String description of parameter. |
| `required` | Yes | Boolean that indicates if this parameter is required. |
| `default` | Optional | Default value. |
| `deprecationMessage` | Optional | Warning message to let users know this parameter is being deprecated and any other relevant information. |

When the action is run on a runner, an environment variable for each input parameter is created. This is the way that the values get transferred to the running applications. The environment variables will have a name like *INPUT_<PARAMETER NAME>* with letters converted to uppercase and spaces replaced with underscores.

The output format for actions is similar. Outputs define data values that the action fills in when it runs. As described in [Chapter 7](ch07.html#ch07), these outputs can be shared with other items in the same workflow.

# GITHUB_OUTPUT

Outputs can also be set in certain cases via redirecting environment variable assignments into *$GITHUB_OUTPUT* on the runner. This is discussed in more detail later in the chapter.

[Table 11-2](#output-parameter-fiel) lists the various fields for an output parameter.

Table 11-2.
Output parameter fields

| Item | Required | Description |
| --- | --- | --- |
| `< output_id >` | Yes | The name of the output must be unique, must start with a letter or _, and can contain only alphanumeric characters, -, or _. |
| `description` | Yes | String description of parameter. |
| `value` | No, if action is a Docker or JavaScript action; yes, if action is a composite action | The value that the output parameter will be mapped to. |

The values of the *input_id* and *output_id* fields are the names that you reference the inputs and outputs by when using the action in your workflow. For example, the steps that follow (taken from the [cache action](https://oreil.ly/0OZ6a) examples) reference the names `path` and `key`—two of the input parameters defined in the separate *action.yml* file I showed earlier, as well as the `cache-hit` output parameter:

```yml
    - name: Cache Primes
      id: cache-primes
      uses: actions/cache@v4
      with:
        path: prime-numbers
        key: ${{ runner.os }}-primes

    - name: Generate Prime Numbers
      if: steps.cache-primes.outputs.cache-hit != 'true'
      run: /generate-primes.sh -d prime-numbers
```

In addition to the pieces mentioned so far, actions can optionally have *branding* attributes. Here’s an example:

```yml
branding:
  icon: 'archive'
  color: 'gray-dark'
```

This lets you select an icon from a subset of the icons at [*feathericons.com*](https://feathericons.com) and a basic set of colors to *brand* your action. You can find a list of all currently supported icons and colors [in the GitHub Actions documentation](https://oreil.ly/3EUjg).

The other key section of the *action.yml* file is the `runs` section. This section specifies what kind of code is used to implement the underlying action and defines any needed execution parameters to run that code, such as the application’s primary file, version of the runtime, etc.

The values specified here vary depending on the way the action is implemented and require more detailed explanations. In the next section, I’ll help you understand the different run environments as I describe each of the different types of actions.

# Types of Actions

GitHub allows actions to be defined in one of three ways:

* As a *composite action* that can be implemented with steps and scripting
* As an action that runs within a Docker container
* As an action that is implemented with JavaScript

Each of these has its advantages and disadvantages. Depending on your use case, one will likely be a better fit than the others. I’ll briefly summarize the characteristics of each type here and then show examples of implementing each type with the important details.

## Composite Action

While it may sound like a combination of the other types of actions and thus more complex, a *composite action* is actually the simplest to implement. So, it makes a good starting point. In the composite action case, the primary difference is that the action’s `run` property (in the *action.yml* file) invokes a list of steps to execute, instead of a program to run. Here’s an example of the property:

```yml
runs:
  using: "composite"
  steps:
```

This `steps` section looks and acts very similar to the `steps` section in a workflow. This provides a couple of immediate advantages for composite actions:

* The `run` section is very easy to read and understand if you’re already familiar with the standard workflow syntax.
* Composite actions can be used to abstract out steps from a workflow into separate actions that can be called from the workflow.

That last point deserves some further discussion. When you are developing workflows over time and creating lots of custom steps, your workflow file can grow significantly. Aggregating some of those steps into a composite action can help make your workflow simpler and more modular. This is similar to the strategy of moving code to functions and procedures in programming languages. So you can think of the *composition* aspect here as being a composition (or collection) of steps moved into their own *subaction*.

A very simple example of how a composite action can be implemented starts with the next listing. This is a very simplistic shell script that simply counts and reports the number of arguments passed in:

```yml
#!/bin/bash

args=($@)
echo ${#args[@]}
```

# Eliminating Empty Arguments

The use of “($@)” in the script is just to eliminate counting empty strings as arguments.

Assume this code is saved in a file named *count-args.sh.* To use this in an action, you need to create an *action.yml* file for it (shown next) that defines and describes the interface to use the script:

```yml
name: 'Argument Counter'
description: 'Count # of arguments passed in'

inputs:
  arguments-to-count:
    description: 'arguments to count'
    required: true
    default: ''

outputs:
  arg-count:
    description: "Count of arguments passed in"
    value: ${{ steps.return-result.outputs.num-args }}

runs:
  using: "composite"
  steps:
    - name: Print arguments if any
      run: |
        echo Arguments: ${{ inputs.arguments-to-count }}.
      shell: bash

    - id: return-result
      run: |
        echo "num-args=`${{ github.action_path }}/count-args.sh  ${{ inputs.arguments-to-count }}`" >> $GITHUB_OUTPUT
      shell: bash
```

The reasons for the format of the first half of the file have already been covered:

* Lines 1 and 2 are just the basic identifying information for the action.
* Lines 3–7 describe the inputs for the action and any default values.
* Lines 8–11 describe the outputs for the action.
* Lines 12–13 describe the type of action this is—its *format*.

Line 14 is where things start to look very familiar with our workflows. Notice that from here until the end of the listing, *action.yml* closely resembles a set of steps that you might find in a job definition in a workflow file. There are three steps defined here, one to print out any incoming arguments, one to compute the number of arguments and set an environment variable with that value, and one to print the result.

The process used in the last step of placing a value into an environment variable and then redirecting it to $GITHUB_OUTPUT is a fairly standard way of returning information from an action of this type. (As discussed in [Chapter 7](ch07.html#ch07), the environment variable $GITHUB_OUTPUT is a standard reference to a temporary file on the runner used to capture and store output from steps.)

These two files are all that’s needed to have a separate action that can be used in other workflows. They can be stored in a separate repository, as shown in [Figure 11-1](#composite-action-layo).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1101.png)<br>
**Figure 11-1. Composite action layout**

You could use this action in a standard workflow with code like the following:

```yml
count-args:
  runs-on: ubuntu-latest
  steps:
    - id: report-count
      uses: skillrepos/arg-count-action@main
      with:
        arguments-to-count: ${{ github.event.inputs.myValues }}

    - run: echo
      shell: bash
      run: |
        echo argument count is ${{ steps.report-count.outputs.arg-count }}
```

In this code, line 52 references the action repository location. You can use the same syntax here to reference an action in any repository location relative to *github.com*.

On line 53, we have the `with` statement, which we know is how arguments can be passed to an action. The `arguments-to-count` piece is the name defined in the *action.yml* `inputs` section. The `github.event.inputs.myValues` reference is a reference to an input for a triggering event defined in the workflow as follows:

```yml
workflow_dispatch:
  inputs:
    myValues:
      description: 'Input Values'
```

In line 57, the reference of `steps.report-count.outputs.arg-count` is referring to the previous step in the workflow with the id of `report-count` that calls the composite action. The `arg-count` portion of this refers to the name of the output item defined in the *action.yml* file.

Another approach to creating an action is to run it in a Docker container. We’ll look at what that implies, and how it is done, next.

## Docker Container Action

A Docker container action is simply, as the name implies, an action that is encapsulated in a Docker container when it is run. There are two approaches that can be used for specifying a container to use in the action: including a Dockerfile that is used to build the container each time or referencing a prebuilt image with the container that can be pulled and executed.

# Caution About Using a Dockerfile

There can be security risks when using a Dockerfile in this way. For that reason, the approach of using a prebuilt image is recommended overall.

Both approaches have a number of advantages. The application is encapsulated in a container image so you have absolute control over all of the setup, environment, runtime, dependencies, etc., that are included in the container. And the implementation can run on any runner that has Docker installed.

GitHub actions have effects on Dockerfile instructions, and you need to be aware of them to ensure you don’t have any surprises during execution. [Table 11-3](#interaction-of-github) summarizes the instructions impacted and the effects.

Table 11-3. Interaction of GitHub Actions and Dockerfile instructions

| Instruction | Limitations | Restrictions/Best practices |
| --- | --- | --- |
| `USER` | Do not use. | Actions must be run by default Docker user: root. |
| `FROM` | Must be first instruction. | Use official images; use version tag, not *latest*; recommended to use Debian OS. |
| `ENTRYPOINT` | Entrypoint defined in *action.yml* overrides ENTRYPOINT in Dockerfile.  Don’t use WORKDIR. If passing args from *action.yml* to Docker container, use shell script through ENTRYPOINT ["/entrypoint.sh"] (script must be executable). | Use absolute path instead of WORKDIR. To have variable substitution in ENTRYPOINT command, use shell format, not exec format.  ENTRYPOINT ["sh”, “-c”, “echo $VARIABLE"] vs. ENTRYPOINT ["echo $VARIABLE"]. |
| `CMD` | Args in *action.yml* will override CMD in Dockerfile. | Document required arguments and omit from CMD; use defaults. |

With those constraints in mind, I’ll look at how to create a basic *Docker container action* using a Dockerfile to build the container each time.

### Running a Docker container action via a Dockerfile

Here’s a generic Dockerfile that can be used to run a simple shell script like the *count-args* script used in the composite action example:

```yml
# Base image to execute code
FROM alpine:3.3

# Add in bash for our shell script
RUN apk add --no-cache bash

# Copy in entrypoint startup script
COPY entrypoint.sh /entrypoint.sh

# Script to run when container starts
ENTRYPOINT ["/entrypoint.sh"]
```

The specification starts with using a minimal operating system image called *alpine* and then adds in *bash* to make running the script easier. After that, I simply copy over a local copy of the script into the image and then set the entrypoint to execute the script on startup.

The contents of *entrypoint.sh* are very similar to the simple *count-args.sh* script I used in the composite example—with one addition. The listing for this file is as follows:

```yml
#!/bin/bash

args=($@)
argcount="${#args[@]}"
echo "argcount=$argcount" >> $GITHUB_OUTPUT
```

The addition is the last line where the file reference $GITHUB_OUTPUT is used to capture output on the runner system (as described in [Chapter 7](ch07.html#ch07)).

# ENTRYPOINT Behavior

Note that this example is using the basic exec form for the Docker ENTRYPOINT instruction. A caveat with using this form is that if you are passing an environment variable in your ENTRYPOINT call, this format will not cause the variable to be interpreted. For example:

```yml
ENTRYPOINT ["echo $MY_VAR"]
```

will not print the value that $MY_VAR resolves to but will instead print “$MY_VAR”. To get variable substitution, you can use the shell form or call the shell directly, as in:

```yml
ENTRYPOINT ["sh", "-c", "echo $MY_VAR"]
```

# Execute Permissions

The *entrypoint.sh* file must be executable. You can use a standard `chmod +x entrypoint.sh` command to modify the permission and make it executable.

The content of the *action.yml* file for the Docker container action is shown in the next listing. This is identical to the *action.yml* file we had for the composite action up until line 11. Since the logic for invoking our script is contained within the Docker process, we don’t need to specify any steps or other direct invocation here, just the inputs, outputs, and how the action is run.

The runs section differentiates this as a Docker container action as opposed to one of the other types, particularly the `using: 'docker'` clause. The *image* section defines the Docker image to use or, in this case, the Dockerfile to use to build that image. Finally, it designates that the inputs are being passed in as arguments to the process running in the container that will be built from the image:

```yml
name: 'Argument Counter'
description: 'Count # of arguments passed in'

inputs:
  arguments-to-count: 
    description: 'arguments to count'
    required: true
    default: ''

outputs:
  arg-count:
    description: "Count of arguments passed in"

runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.arguments-to-count }}
```

We can complete this action with a simple *README.md* file:

```yml
# Count arguments docker action
This action prints out the number of arguments passed in

## Inputs
## `arguments to count`

**Required** The arguments to count.

## Outputs
## `argcount`

The count of the arguments.

## Example usage
```yaml
uses: <repo>/arg-count-docker-action@v1
with:
  arguments: <arguments>
\```
```

### Running a Docker container action via an image

Another option for running a Docker container action is using an existing Docker image. This option is very similar to the version that uses the Dockerfile, with the following differences:

* An image is built from the Dockerfile in advance of using the action and pushed to a repository.
* The image location (rather than *Dockerfile*) is specified in the *action.yml* file.

For this example, an image has been built from the Dockerfile used in the last section, tagged as *quay.io/techupskills2/arg-count-action:1.0.1* and pushed out to the *quay.io* registry. To make the image available as an action, I just need to change the *action.yml* file. Here’s an *action.yml* file for this use case:

```yml
name: 'Argument Counter'
description: 'Count # of arguments passed in'

inputs:
  arguments-to-count:
    description: 'arguments to count'
    required: true
    default: ''

outputs:
  arg-count:
    description: "Count of arguments passed in"

runs:
  using: 'docker'
  image: 'docker://quay.io/techupskills2/arg-count-action:1.0.1'
  args:
    - ${{ inputs.arguments-to-count }}
```

The only line different here versus *action.yml* for the Dockerfile version is line 13. In this case, I specify the image to use instead of a Dockerfile. Notice that I had to add the `docker://` syntax at the start of the line. With this change, I can use a predefined image for running the action code. While this does save the overhead of building the image from scratch, it does incur the cost of downloading the image (if it is not already on the system).

# Required Image Permissions

Currently, to use an image in an action this way, it must be publicly accessible. While there is a Docker login action that can be used to authenticate to a registry, GitHub Actions will try to download the image used at the start of any workflow using the action with the image. This timing issue means that it will not wait for authentication to happen, even if the login process is a different job.

Finally in this section on action types, we’ll look at how to create a custom GitHub action written in JavaScript.

## Creating a JavaScript Action

If you are proficient in programming and want to be able to code a more complex action without incurring the overhead of using Docker, GitHub Actions provides the flexibility to code the action logic in JavaScript. If you go this route, there are a few recommended best practices:

* To keep your code compatible with the different types of GitHub-hosted runners, the code should be pure JavaScript (not relying on any other binaries).
* Download and install Node.js 16.x (as of the time of this writing).
* For faster development, utilize these packages from the GitHub Actions Toolkit module for Node.js. (The Actions Toolkit is discussed more later in this chapter.)
  + *@actions/core*: provides interfaces to workflow commands, input/output variables, exit values, etc.
  + *@actions/github*: provides a REST client to use with GitHub Actions contexts access

For the last bullet, you can install the toolkit packages via these commands:

```sh
npm install @actions/core
npm install @actions/github
```

After that, you’ll have a local *node_modules* directory that has the modules just installed and a *package-lock.json* file that has the dependencies and versions for each installed module. These will need to be committed with the rest of your code into a GitHub repository for your action.

To illustrate creating a JavaScript action, I’ll continue with using the *argument counter* example. Here’s the code for the *action.yml* file:

```yml
name: 'Argument Counter'
description: 'Count # of arguments passed in'

inputs:
  arguments-to-count:
    description: 'arguments to count'
    required: true
    default: ''

outputs:
  argcount:
    description: "Count of arguments passed in"

runs:
  using: 'node22'
  main: 'index.js'
```

The primary difference between this *action.yml* specification and the ones for the other types of actions are the last two lines under the `runs` section. These specify that this is a JavaScript action via the line that specifies the version of *node* being used and the main JavaScript file. This is the information that the action runner needs to know to start running this action.

The actual code for the *index.js* file is next:

```js
// simple demo file for javascript github action
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `arguments-to-count` input defined in action metadata file
  const inputArgs = core.getInput('arguments-to-count');
  console.log(`Arguments = ${inputArgs}!`);

  const argCount = inputArgs.split(/\s+/).length;
  core.setOutput("argcount", argCount);

  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2);
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```

The code itself is pretty straightforward, but a few lines warrant some additional comments:

* Lines 2–3 pull in the Actions Toolkit modules.
* Line 7 references the input parameter defined in the *action.yml* file.
* Line 10 references the output parameter defined in the *action.yml* file.
* Line 12 shows a way to use the Actions Toolkit GitHub module to get info about the event that triggered the workflow.
* Line 15 uses the Actions Toolkit core module to log an error message and set a failed exit code if an error is thrown during execution of the code.

To complete the set of files for this action, you can add a README:

```yml
# Count arguments javascript action

This action prints out the number of arguments passed in

## Inputs
## `arguments-to-count`

**Required** The arguments to count.

## Outputs
## `argcount`

The count of the arguments.

## Example usage

```yaml
uses: <repo>/arg-count-javascript-action@v1
with:
  arguments: <arguments>
```yml
```

# TypeScript Actions

You can also create a JavaScript action using TypeScript. GitHub provides [a template for creating a TypeScript action](https://oreil.ly/6U2VC) that includes support for compiling, testing, validating via a framework, publishing, and versioning.

# Completing Your Action Creation

Once you have your code, *action.yml*, and other files created, you add them to the repository in the standard add/commit/push way. However, there is still one important item to do to make your action easily usable over time—adding a tag.

When your action is used in a workflow, a particular version can be accessed by any valid Git reference—a commit SHA1, a branch name, or a tag. For example, if we want to use the action *gradle/gradle-build-action*, we could currently reference it via the following:

```yml
uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
uses: gradle/gradle-build-action@main
uses: gradle/gradle-build-action@v3
uses: gradle/gradle-build-action@v3.5.0
```

While not a requirement, most actions that are intended to be used by others conform to a regular tagging and release strategy. This means that as new versions of the action code are created, they are tagged in the GitHub repository, and then a release is made in the repository:

```sh
git tag -a -m "Description of this release"
git push --follow-tags
```

The usual convention is to start tags with a *v* and to keep a regular tagging strategy using [semantic versioning](https://semver.org), with the MAJOR.MINOR.PATCH format. As an ease-of-use convention, tags with only the MAJOR portion (v1, v2, etc.) are usually maintained and moved to the most recent MAJOR.MINOR.PATCH version as new ones are created. For example, if the *v3* tag currently points to my version *v3.2.3* and I now create *v3.2.4*, then the expectation would be that the *v3* tag is moved (deleted and re-created) sometime in the near future to point to *v3.2.4*.

You may also see tags that reference release candidates or beta versions, such as *v3.0-rc.2* or *v3.0-beta.1*. In many cases, users just reference the major version tag of an action they want to use, as in `uses: gradle/gradle-build-action@v3`.

# Shortcut for Using Latest Version of an Action

In actions that are available on the [Actions Marketplace](https://oreil.ly/EXxYi), you can typically find the latest MAJOR.MINOR.PATCH version identified near the top of the main page. For example, note the 2.4.2 and *Latest version* identifier at the top of the gradle-build-action page in [Figure 11-2](#info-on-latest-versio22).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1102.png)<br>
**Figure 11-2. Info on latest version at top of Marketplace page**

Also, there is a large green Use latest version button at the right of most Marketplace action main pages ([Figure 11-3](#selecting-a-version-o)). Clicking the arrow in the right part of that button will give you a list of previous versions to select from.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1103.png)<br>
**Figure 11-3. Selecting a version of a public action**

Conveniently, clicking the main part of that button will provide you with a code sample that you can copy and paste into your workflow to use the action, as shown in [Figure 11-4](#getting-the-code-to-u11). This will be the most basic usage of the action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1104.png)<br>
**Figure 11-4. Getting the code to use the latest action version**

# Publishing Actions on the GitHub Marketplace

So far in this chapter, we’ve looked at how to create and use actions from your own repositories and from the Actions Marketplace but not how to connect the two. If you want to share your action more widely with the GitHub community, putting it on the Marketplace is a good option—with some preparation.

Per the GitHub Actions documentation (as of the time of this writing), actions aren’t reviewed by GitHub and can be published to the GitHub Marketplace immediately if they meet the following requirements:

* The action must be in a public repository.
* Each repository must contain a single action.
* The action’s metadata file (*action.yml* or *action.yaml*) must be in the root directory of the repository.
* The name in the action’s metadata file must be unique.

  + The name cannot match an existing action name published on GitHub Marketplace.
  + The name cannot match a user or organization on GitHub, unless the user or organization owner is publishing the action. For example, only the GitHub organization can publish an action named github.
  + The name cannot match an existing GitHub Marketplace category.
  + GitHub reserves the names of GitHub features.

If you have a public repository with an action metadata file and you are logged in as the owning user, then you should see a banner at the top of the Code tab that looks like [Figure 11-5](#banner-with-option-to).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1105.png)<br>
**Figure 11-5. Banner with option to draft release for getting action to Marketplace**

Clicking the Draft a release button then takes you to a screen to fill out information to create the draft release for the Marketplace, including selecting the version of code to share, as shown in [Figure 11-6](#screen-to-draft-a-rel).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1106.png)<br>
**Figure 11-6. Screen to draft a release**

From the *Choose a tag* drop-down, you can choose an existing tag or create a new one. Guidance for picking a tag is shown in the *Tagging suggestions* to the right. (Basically, ensure that your tag starts with a *v* and follows semantic versioning guidelines.) You also have the option of choosing from a particular branch or recent commit.

You must click, read, and accept the *accept the GitHub Marketplace Developer Agreement* link before publishing an action. This is the gateway to ensuring that you have met the basic requirements for your action to get on the Marketplace. It also provides suggestions that you might want to add on to your action like an Icon or Color, if it sees they are missing ([Figure 11-7](#pre-release-checks-on)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1107.png)<br>
**Figure 11-7. Prerelease checks on the current state of the action**

Items like the Icon and Color are optional. Having a basic README is re­quired. Ideally, to be most usable, the README should have similar information (at least as far as inputs and outputs) as the action metadata file included in it.

After updating any needed/desired information from this review, you can go ahead and fill out the rest of the information in terms of categories, release notes, etc. [Figure 11-8](#filling-in-final-info) shows examples of doing this for the demo action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1108.png)<br>
**Figure 11-8. Filling in final information prior to release**

Once that has been completed and the Publish release button is clicked, you’ll see the updated release available with a Marketplace button next to the title ([Figure 11-9](#release-available)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1109.png)<br>
**Figure 11-9. Release available**

If you click the Marketplace button, then you should be able to see the action on the public Marketplace ([Figure 11-10](#initial-release-of-ac)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1110.png)<br>
**Figure 11-10. Initial release of action on Marketplace**

After completing this publish to the marketplace, there will also be a *Use this GitHub Action with your project* banner displayed on your repository when on the *Code* tab ([Figure 11-11](#banner-option-to-use)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1111.png)<br>
**Figure 11-11. Banner option to use the new action with your project**

## Updating Actions on the Marketplace

Updating actions on the Marketplace follows the same general process as for any other updates. There is a reference to the underlying GitHub repository on the action’s Marketplace page. Another user can go to that page, fork the project, and make a pull request on that repository. Then, as the owner of the repo, you can review the pull request and decide whether to accept it. In this example, my *README.md* file is rather sparse as it currently stands on the Marketplace. Suppose another user notices and submits a PR to update it. Then I can merge it, as shown in [Figure 11-12](#updates-for-readme-fo).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1112.png)<br>
**Figure 11-12. Updates for README for Marketplace action**

After the merge, my action on the Marketplace is automatically updated to reflect the change (see [Figure 11-13](#updated-marketplace-p)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1113.png)<br>
**Figure 11-13. Updated Marketplace page**

# Choosing the README Content

The README content on the Actions Marketplace page will be displayed from the default branch of the associated repository. As well, the link to the code repository will take the user to the default branch. If you want to use a different branch as the basis for your public action, make sure to set the default branch appropriately in the repository.

## Removing an Action from the Marketplace

If you want to remove an action from the Marketplace and you are the owner, you can simply select the Delist button in the upper right of the marketplace page ([Figure 11-14](#button-to-remove-acti)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1114.png)<br>
**Figure 11-14. Button to remove action from Marketplace**

You’ll be prompted for confirmation. After the action is delisted, you’ll have the option again in the repository to draft a release and publish it back to the Marketplace if you wish.

In the course of explaining how you create the different kinds of actions, I’ve mentioned the GitHub Actions toolkit multiple times. Before ending this chapter, I’ll talk a bit more about what the toolkit is, what it provides, and how you can leverage it.

# The Actions Toolkit

To help make creating GitHub actions easier for JavaScript actions and via workflow commands (discussed in the next section), there’s the GitHub Actions Toolkit. This is a set of node modules that you can install with the node package manager (npm), and immediately have easy ways to work with Actions features. The current (as of the time of this writing) toolkit packages and their purpose are listed in [Table 11-4](#list-of-available-act).

Table 11-4. List of available actions packages

| Package | Purpose |
| --- | --- |
| @actions/core | Provides functions for inputs, outputs, results, logging, secrets and variables |
| @actions/exec | Provides functions to exec cli tools and process output |
| @actions/glob | Provides functions to search for files matching glob patterns |
| @actions/http-client | A lightweight HTTP client optimized for building actions |
| @actions/io | Provides disk i/o functions like cp, mv, rmRF, which, etc. |
| @actions/tool-cache | Provides functions for downloading and caching tools, e.g., setup-* actions |
| @actions/github | Provides an Octokit client hydrated with the context that the current action is being run in |
| @actions/artifact | Provides functions to interact with action artifacts |
| @actions/cache | Provides functions to cache dependencies and build outputs to improve workflow execution time |

Making these available to use in your JavaScript code is as simple as `npm install <name-of-package>`, for example:

```sh
npm install @actions/core
```

You can then use the functions from the particular package in your code:

```js
core.setOutput("argcount", argCount);
```

Per the earlier example, you need to be sure to add the generated modules to your GitHub repository.

# Using Action Toolkit Packages with TypeScript

As noted previously, there is a way to create a JavaScript action using TypeScript. If you are doing this and need to use a package from the Actions toolkit, then you would need to bring it in via `import * as <reference> from <package-name>;`. Here is an example:

```js
import * as core from '@actions/core';
```

In a Docker container action, you can just base your image off of one with node and execute a step in the Dockerfile to `npm install` the appropriate package.

## Using Workflow Commands from the Toolkit

The actions toolkit includes several functions that can be executed through workflow commands. Workflow commands are a way for actions to set output values, environment variables, etc., on runner machines. (Some of these have been previously discussed in other chapters.) These commands primarily use a form like this:

```yml
echo "::workflow-command param1={data},param2={data}::{value}"
```

As an example, to print a warning message in JavaScript, you could use the following: `core.warning("No timeout supplied!");`.

You could do the following in YAML:

```yml
- id: return-result
  run: |
    echo "::warning::No timeout supplied"
```

This is an example of a step printing a warning message via a workflow command.

# Additional Parameters

Some workflow commands include the ability to add other parameters to identify specific locations within workflow code where an issue occurs. For example, with the warning workflow command, you can also supply a filename, line, starting column, and ending column:

```js
echo "::warning file=abc.js, line=10::Missing key"
```

As of the time of this writing, there seem to be instances when not all of these types of parameters work as expected.

[Table 11-5](#mappings-between-tool) is taken largely from the GitHub Actions documentation and shows the equivalent workflow command for a subset of available toolkit functions.

Table 11-5. Mappings between toolkit actions and workflow commands

| Toolkit function | Equivalent workflow command |
| --- | --- |
| `core.addPath` | Accessible using environment file `GITHUB_PATH` |
| `core.debug` | `debug` |
| `core.notice` | `notice` |
| `core.error` | `error` |
| `core.endGroup` | `endgroup` |
| `core.exportVariable` | Accessible using environment file `GITHUB_ENV` |
| `core.getInput` | Accessible using environment variable `INPUT_{NAME}` |
| `core.getState` | Accessible using environment variable `STATE_{NAME}` |
| `core.isDebug` | Accessible using environment variable `RUNNER_DEBUG` |
| `core.summary` | Accessible using environment variable `GITHUB_STEP_SUMMARY` |
| `core.saveState` | Accessible using environment file `GITHUB_STATE` |
| `core.setCommandEcho` | `echo` |
| `core.setFailed` | Used as a shortcut for `::error` and `exit 1` |
| `core.setOutput` | Accessible using environment file `GITHUB_OUTPUT` |
| `core.setSecret` | `add-mask` |
| `core.startGroup` | `group` |
| `core.warning` | `warning` |

# Command Special Characters

When using a workflow command as previously described, the entire command must be on a single line. Any special characters that might cause parsing issues must be URL encoded. [Table 11-6](#encodings-for-special) shows the characters and the required encodings.

Table 11-6. Encodings for special characters when used in workflow commands

| Character | Encoded value |
| --- | --- |
| `%` | `%25` |
| `\r` | `%0D` |
| `\n` | `%0A` |
| `:` | `%3A` |
| `,` | `%2C` |

Here is an example: `echo "::warning::Line 1%0ALine 2"`.

There are several common use cases for leveraging workflow commands. These include the following:

* Displaying messages with different severity levels
* Grouping log data via the `group` and `endgroup` commands—this creates an expandable section for less scrolling
* Masking sensitive information so it doesn’t show up in the log

More information and examples of these use cases can be found in [“Augmenting and Customizing Logging”](ch10.html#augmenting_customizing).

As you can see, there is a lot of utility provided by the Action Toolkit packages and the workflow commands. These are the missing pieces to fill in the functions that seem simple but would otherwise require you to do custom coding.

Finally, in this chapter, I’ll describe a way to create and store actions more directly—from within an existing repository.

# Local actions

You may think of actions as always being defined in a separate, independent repository. While that is the most common case, you can also create actions within any repository and access them in the same relative way as any other content in the same repository. These can be called *local actions*.

As an example, suppose you want to create an action within your existing repository that does some basic testing on code changes in the repository but it’s not significant enough to create a separate repository for the action.

For the simple use case here, I’ll reuse some demo code that I use in one of my classes. To facilitate making this a local action, store it in a *.github/actions* directory instead of a *.github/workflows* directory. Within the *.github/actions* directory, I’ll create a directory for the action called *test-action*. The demo code is in a file called *test-script.sh*. [Figure 11-15](#creating-a-local-acti) shows saving the code in the local actions area of the repository.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1115.png)<br>
**Figure 11-15. Creating a local action in the repository**

After this, in order to make this script usable as an action, I follow the same process detailed previously and create an *action.yml* file in the same directory (*greetings-ci/.github/actions*) and save it ([Figure 11-16](#action-yaml-file-for)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1116.png)<br>
**Figure 11-16. action.yaml file for local action**

Notice that there is nothing special about this *action.yml* file—it follows the standard pattern and requirements. The trick is just to make sure it is stored in the same directory as the script.

Once those files are created and saved, the local action is ready to use. An example usage, based on the defined arguments it expects, might look like the following:

```yml
- uses: actions/checkout@v4

- name: run-test
  uses: ./.github/actions/test-action
    with:
      artifact-version: ${{ needs.build.outputs.artifact-tag ||
 github.event.inputs.myVersion }}
      arguments-to-print: ${{ github.event.inputs.myValues }}
```

Take note of how the local action is referenced in the preceding example. It is referenced by the relative path to the directory for the action in the same repository: *./.github/actions/test-action*.

# Reusable Workflows

Besides creating and referencing actions locally, you can also create and call workflows in the same repository if they are created as *reusable workflows* (discussed in [Chapter 12](ch12.html#ch12)). The syntax is the same as for referencing a local action, for example:

```yml
jobs:
  invoke-workflow-in-this-repo:
    uses: ./.github/workflows/my-local-workflow.yml
```

When called this way, the workflow that is invoked will have the context from the same commit as the caller.

# Conclusion

In this chapter, we’ve looked at what it takes to create your own custom GitHub action. You can create the logic for your action by creating a composite action that consists of running steps similar to a workflow, crafting a Dockerfile or Docker image to run an action in a container, or coding more complex logic with JavaScript. GitHub Actions provides a set of toolkit functions and workflow commands to help with implementation.

What makes any of these implementations usable as an action is the *action.yaml* file. This file has a specific format that defines the inputs, outputs, run configuration, and, optionally, branding info. Once created, the actions can be placed on the GitHub Actions Marketplace if desired.

Just as you can define different types of actions and create custom functionality within them, you can also do the same with workflows. [Chapter 12](ch12.html#ch12) will explain several more advanced techniques and use cases for those.

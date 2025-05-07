# Chapter 12. Advanced Workflows

By this point in the book, you’ve seen many basic examples of workflows. Beyond the basics are several approaches for leveraging workflows that can greatly simplify repeated use. In this chapter, I’ll show you several ways you can leverage workflows to get additional flexibility and reuse.

In particular, I’ll cover implementation and use patterns for the following:

* Starter workflows
* Reusable workflows
* Required workflows

# Creating Your Own Starter Workflows

Starter workflows were introduced in [Chapter 1](ch01.html#ch01). As a reminder, starter workflows are basic workflow examples, tailored for a particular purpose, that anyone can use as initial code when you need to create a new workflow. As of the time of this writing, the ones provided with GitHub Actions fall into several categories:

> *Automation*
>> Helpful code for doing automated processing such as handling pull requests
>
> *Continuous Integration*
>> Monitoring code changes and initiating follow-on processes such as building and testing
>
> *Deployment*
>> Using automation to publish and deploy software updates
>
> *Security*
>> Adding security automation, such as code scanning, dependency review, etc., to your workflows
>
> *Pages*
>> Automating deploying and packaging GitHub Pages sites using different technologies

##### More About Categories

Although only a few high-level categories may be shown when selecting starter workflows, there is a more extensive set of categories in the [GitHub Actions repository](https://oreil.ly/CWqvl).

These can be further refined with the [list of languages](https://oreil.ly/_Fko7) and [tech stacks](https://oreil.ly/yMDXT) known to GitHub.

When you are looking at the available templates, templates that more closely match the type of content in your repository will feature more prominently.

The set of starter workflows will show up when you select the Actions tab in a repository and you have no existing workflows. If you already have workflows and want to see the starter page again, you can click the New workflow button in the Actions tab or go to *github.com/<repo path>/actions/new*.

For very basic tasks, such as a simple build of a project, the starter workflows that are provided by GitHub may be all the code you need. However, in most cases, you will want to modify/extend them to better suit your needs. If you find yourself doing this repeatedly with the same kind of changes, that can be an indication that you and your team or organization could benefit from having a custom starter workflow.

Creating a custom starter workflow involves three main tasks:

* Create the starter workflow area (if it doesn’t already exist).
* Create and add the code for the initial workflow.
* Create and add supporting pieces.

The ways to do these tasks are covered in the next few sections.

# Permissions to Create Workflows

Only organization members that have permission to create workflows will be able to use your custom starter workflows.

## Creating a Starter Workflow Area

Creating a starter workflow area means creating a central location to store your starter workflows where others will be able to access them as well. To be able to create this area, you need to first ensure you have write access to the GitHub organization.

Next, if it doesn’t already exist, create a new public repository called *.github* in your organization. In this .github repository, create a *workflow-templates* directory.

# .github Repository

If you’re not familiar with the .github repository, it has other uses besides a place for starter workflows. It has traditionally been used as a top-level holder for files that apply across an organization, like a public *README* file placed in a directory called *profile* that describes more about the organization.

Now you’re ready to add the initial code for your starter workflow.

## Creating a Starter Workflow File

Creating a starter workflow is just creating a workflow file with the same elements as any other workflow—an `on` section, `jobs` section, and so on. You should think about what you want your starter workflow to do/automate for your users. And then consider how you can best enable that functionality dynamically and generically. Then you code it.

Since you want this to be a *template* of sorts and usable in different repositories, you may want a way to allow for automatic substitution of some values. There are a few built-in variables that can be used in your starter workflow and will be replaced automatically. These are listed next, with a description of the values that will be filled in when used:

* `$default-branch`: will substitute the branch from the repository, for example, `main` or `master`
* `$protected-branches`: will substitute any protected branches from the repository
* `$cron-daily`: will substitute a valid but random time within the day

Of these, the `$default-branch` is probably the most useful for substitution. You can use that in the `on` clause for events like `push` and `pull_request` that often trigger on the default branch.

As an example of creating a starter workflow, I’m going to create a simple one for an organization I have called *rndrepos*. This workflow will gather and report some very basic info about any repository that uses it in the organization—the size of the cloned repository and the GitHub context for the repository.

The starter workflow will be named *Repo Info with context*. The main code is shown in the next listing:

```yml
name: Repo Info with context

on:
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]

jobs:
  info:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Show repo info
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: |
          echo Repository is ${{ github.repository }}
          echo Size of local repository area is
          du -hs ${{ github.workspace }}
          echo Context dump follows:
          echo "$GITHUB_CONTEXT"

# add your jobs here
```

Given what you already know about workflows from the preceding chapters of the book, this code should be pretty straightforward. There’s a single job that contains two steps. The first step checks out the code. The second runs a shell command on the runner to determine the size of the cloned repository and then dumps out the context.

The only thing that is really different here from previous examples is the use of the `$default-branch` instead of a hard-coded value or pattern. Per the previous explanation, it will be filled in with the actual value for the default branch when this starter workflow is configured for use with a repository.

Once this code is created, you’re ready to complete the setup of the starter workflow itself.

## Adding Supporting Pieces

To complete the setup of this code as a starter workflow, I place it in the .github repository at the root of the organization, in the *workflow-templates* subdirectory. There are two other files I include with the workflow file itself:

* An *.svg* graphics file that has the icon I want to have show up with this workflow in the starter page.
* A metadata file that has the same name as the workflow file but with a *.properties.json* extension. This is the file that contains the metadata that GitHub Actions needs to be able to surface the *rndrepos-info.yml* as an actual starter workflow.

These files should also be stored in the *workflow-templates* directory.

The *.svg* file I added is named *check-square.svg*. See the sidebar for more information on *.svg* files in general.

For the metadata file, since I named the actual workflow file *rndrepos-info.yml,* the corresponding properties file is named *rndrepos-info.properties.json*. [Figure 12-1](#files-that-make-up-th) shows the files in the *workflow-templates* directory of the *.github* repo in the *rndrepos* organization.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1201.png)<br>
**Figure 12-1. Files that make up the starter workflow**

##### SVG Files

SVG stands for *Scalable Vector Graphics* and is the format for the icon files you can use with your starter workflow. While you can easily create your own, you can find ready-made SVG files on the web.

The content of the *rndrepos-info.properties.json* file is shown in the next listing:

```yml
{
  "name": "RndRepos Info Workflow",
  "description": "RndRepos informational starter workflow.",
  "iconName": "check-square",
  "categories": [
    "Text"
  ],
  "filePatterns": [
    ".*\\.md$"
  ]
}
```

Because this is a JSON file, the syntax is different from what you usually see when working with workflow-related files. However, it is still fairly easy to read and interpret.

Lines 2–4 are basic metadata about the workflow. The *name* and *description* fields are required.

Lines 5–7 compose an optional section that describes the programming *language* category of the workflow. When a user is looking at starter workflows for a repository, if GitHub detects that the repository contains files associated with this language, it will feature this workflow more prominently in the list. (See [the documentation](https://oreil.ly/sj1TE) for the list of languages and associated files that you can choose from.) For illustration purposes, I’m just setting this to *Text* to be generic.

Lines 8–10 are another optional section that defines file patterns to check for in the root of the repository. This is used as a way to check if the workflow should apply to a given repository.

With all of this content created and in the *.github/workflow-templates* directory, the starter workflow is ready to use.

## Using the New Starter Workflow

With the items in the previous sessions done, the new starter workflow is available for repositories in the *rndrepos* organization. If you click the *Actions* tab in a repository with no existing workflows or select the New Workflow button from the *Actions* tab, you will see the starter workflow, as shown in [Figure 12-2](#starter-workflow-show).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1202.png)<br>
**Figure 12-2. Starter workflow showing up in list**

You can now click the Configure button and get the custom workflow automatically populated into your repository, as shown in [Figure 12-3](#new-workflow-in-repo).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1203.png)<br>
**Figure 12-3. New workflow in repo based off custom starter workflow**

Notice that the `$default-branch` placeholder has been updated to have the actual default branch from the repository: `main`.

Of course, you can make starter workflows as simple or as complex as needed. Keep in mind, though, that if you are making a starter workflow more complex, it may not be well suited to being a *starter* workflow. You might be better off moving some of the functionality to be housed in a custom action or a *reusable workflow*, the topic of the next section.

# Reusable Workflows

Code reuse is one of the main tenets of good design and development. The ability to provide a set of code that can be reused by multiple callers, easily and without modification, is a sign of an effective design and execution strategy.

This also holds true for GitHub Actions workflows. If you find yourself creating useful workflow code that is needed by (or may be needed by) multiple other workflows, you should consider separating out that code into a separate, *reusable* workflow. Fortunately, that is simple to do.

With GitHub Actions, you can make a workflow into a reusable workflow by adding the special trigger event `workflow_call`. `workflow_call` indicates that a workflow is callable from another workflow. The reusable workflow gets the event payload of the calling workflow.

##### Difference Between workflow_call, workflow_dispatch, and workflow_run

Among the set of events that can trigger workflows, there are several that start with *workflow_* and sound/look similar. Here’s some quick info to disambiguate them:

* `workflow_call` is used to make a workflow callable from another workflow. When a workflow has this event in it, it is considered *reusable.* When called from another workflow, it will get the full event payload from the calling workflow.
* `workflow_dispatch` provides a way to manually trigger a workflow. When this event is present, the workflow can be manually triggered via the GitHub API, the GitHub CLI, or the Actions browser interface.
* `workflow_run` allows you to trigger execution of a secondary workflow after a prerequisite workflow has run to completion—whether it succeeded or failed.

As an example, I’ll change the starter workflow used in the previous section into a reusable workflow. As with other workflows, reusable workflows reside in a *.github/workflows* directory. (Subdirectories underneath that are not supported.) They can be in the same repository *or* in any repository that is accessible to the organization or enterprise, assuming you have set the accesses appropriately.

To create the reusable workflow and make it more generally available, I’ll create a new repository in the organization called *common.* Within that repository, I’ll create a reusable version of the starter workflow with the following contents and save it as *rndrepos/common/.github/workflow/repo-info.yml*:

```yml
name: Repo Info with context

on:
  workflow_call:

jobs:
  info:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Show repo info
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: |
          echo Repository is ${{ github.repository }}
          echo Size of local repository area is
          du -hs ${{ github.workspace }}
          echo Context dump follows:
          echo "$GITHUB_CONTEXT"
```

The main difference here versus other workflows is the use of the `workflow_call` event trigger in lines 3–4. This makes the workflow callable from others.

Here’s an example of a caller workflow in a separate repository in the same organization (*rndrepos/wftest/.github/workflows/get-info.yml*):

```yml
name: Get Info

on:
  push:

jobs:
  info:
    uses: rndrepos/common/.github/workflows/repo-info.yml@main
```

There are a couple of details worth pointing out in the caller workflow. First, note that the workflow follows the same standard format until we get down to the job definition for `info`. Second, notice that the `uses` clause is referenced directly in the job definition, not within a step. In fact, the job does not have any steps in this case. Finally, the path in the `uses` statement is what invokes the reusable workflow at that path. It is fully qualified—all the way to the individual file within the repository. (In this case, it references the version current on the *main* branch, but it could just as well be a tag or SHA value.)

For calling reusable workflows to succeed, accesses must be set appropriately. There are several options for being able to call another workflow, but at least one of the following must be true (as taken from the GitHub documentation):

* Both workflows are in the same repository.
* The called workflow is stored in a public repository, and your organization allows you to use public reusable workflows.
* The called workflow is stored in a private repository, and the settings for that repository allow it to be accessed.

In general, you can set up needed access via the *organization/enterprise settings* for actions. See the section in [Chapter 9](ch09.html#ch09) on setting permissions to run/use workflows in repositories for more details on how to enable access.

This shows the basic use case for a reusable workflow, but what if you want or need to pass other parameters, such as user inputs or secrets? Fortunately, that’s easy to do.

## Inputs and Secrets

In the listing that follows, I have defined a reusable workflow to create a GitHub issue. To make this flexible, the workflow takes two strings, title and body, as input parameters. It also requires a personal access token to be passed in via a secret since the reusable workflow itself exists in a separate repository:

```yml
name: create-repo-issue

on:
  workflow_call:
    secrets:
      token:
        required: true
    inputs:
      title:
        description: 'Issue title'
        required: true
        type: string
      body:
        description: 'Issue body'
        required: true
        type: string

jobs:
  create_issue:
    runs-on: ubuntu-latest

    permissions:
      issues: write

    steps:
      - name: Create issue using REST API
        run: |
          curl --request POST \
          --url https://api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.token }}' \
          --header 'content-type: application/json' \
          --data '{
            "title": "${{ inputs.title }}",
            "body": "${{ inputs.body }}"
          }' \
          --fail
```

At line 4, there is the `workflow_call` event trigger for a reusable workflow. Underneath that, notice that there is a dedicated `secrets` section. In this case, we are simply passing in a single secret containing a personal access token to use for creating the issue (line 30). This is followed by the declarations for the `inputs` that pass in the title and body to use for the new issue. Notice that both of these require the `type` parameter as part of the declaration and that they are simply dereferenced as `inputs.<name of parameter>`.

The next listing shows a simple workflow that calls the reusable workflow previously defined. This assumes that a secret named PAT has been created in the calling repository containing the personal access token that will be used to create the issue:

```yml
name: Create demo issue

on:
  push:

jobs:

  msg:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Simple demo for reusable workflow"

  issue:
    uses: rndrepos/common/.github/workflows/create-repo-issue.yml@main
    secrets:
      token: ${{ secrets.PAT }}
    with:
      title: "Test issue"
      body: "Test body"
```

# Inheriting Secrets

To simplify passing secrets, the calling workflow can just specify `secrets: inherit`, and the reusable workflow will not have to declare any secrets passed in. The reusable workflow still must know the name of the secret to use.

Here is an example of calling a reusable workflow with the inherit option:

```yml
  issue:
    uses: rndrepos/common/.github/workflows/create-repo-issue.yml@main
    secrets: inherit
    with:
      title: "Test issue"
      body: "Test body"
```

Notice in this workflow that the `issue` job consists only of the `uses` call to the reusable workflow. There is a separate job (`msg`) to print out the informational text. This illustrates one of the constraints around using reusable workflows. A workflow job can either call a reusable workflow or have a series of steps. If it has a series of steps, then it will need the `run-ons` statement. If it is just using a reusable workflow, then it will not.

## Outputs

You can also return values from a reusable workflow. Inside of a step within a job in the reusable workflow, you assign a return value to an environment variable and direct that to `$GITHUB_OUTPUT`. Then you can create an `outputs` section for the job that captures the value from the step. Finally, you would have an `outputs` section for the `workflow_call` trigger that would return the value from the job. (This is the same process outlined for returning outputs in [Chapter 7](ch07.html#ch07).)

As an example, here’s a listing for the previous reusable workflow that has been modified to return the number of the new issue that has been created:

```yml
name: create-repo-issue3

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
      issue-num:
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

If you look at line 32, the code is running the API call to create the issue. At line 41, it parses the JSON output through the *jq* query tool to get the number id of the new issue. At line 42, it sets the variable *inum* to the resulting value.

Back up at line 24, that output value from the step is captured in an output value with the same name at the level of the job. And, in lines 15–17, I define an output parameter at the level of the workflow to be able to capture and return the value.

In the caller workflow, the code would look something like the next listing:

```yml
name: Create demo issue 3

on:
  push:

jobs:
  create-new-issue:
    uses: rndrepos/common/.github/workflows/create-issue.yml@v1
    secrets: inherit
    with:
      title: "Test issue"
      body: "Test body"

  report-issue-number:
    runs-on: ubuntu-latest
    needs: create-new-issue
    steps:
      - run: echo ${{ needs.create-new-issue.outputs.issue-num }}
```

The first job starting at line 7 invokes the reusable workflow with the `use` statement and parameters. The second job starting at line 14 simply echoes out the returned issue number on line 18. The name of the output parameter—`issue-num`—is the same one that was declared at the workflow level in the reusable workflow.

## Limitations

As of the time of this writing, there are a few limitations around reusable workflows:

* While you can call a reusable workflow from another reusable workflow, you can only do this nesting to a depth of four calls.
* A caller workflow can call a maximum of 20 reusable workflows. Nested workflow calls count toward this limit as well.
* Environment variables set in an `env` context in the caller workflow are not propagated to the called workflow.
* You can’t reference a reusable workflow that’s in a separate private repository. Only workflows in the same private repository can reference a reusable workflow in that repository.

In addition to the limitations of reusable workflows, one of the other challenges that users often run into is understanding how reusable workflows differ from composite actions in GitHub Actions. See the following sidebar for a further explanation of the differences.

# Comparison to Composite actions

Composite actions provide a way to create a custom action that encapsulates multiple steps. They allow for reuse of a set of steps by being called as a separate action and so are convenient for grouping together steps that need to be reused. (Composite actions are discussed in detail in [Chapter 11](ch11.html#ch11).) But they differ from the kind of reuse you can get from reusable workflows in several ways, as shown in [Table 12-1](#reusable-workflows-vs).

Table 12-1.
Reusable workflows versus composite actions

| Reusable workflows | Composite actions |
| --- | --- |
| Can have up to 4 nested calls to reusable workflows | Can have up to 10 nested composite actions in a workflow |
| Able to pass in secrets directly | Must pass in secrets as an input |
| Can use *if* conditionals | Cannot use *if* conditionals |
| Can be managed as simple YAML files in an existing repository | Require their own independent repository |
| Can have multiple jobs | Can only have steps that equate to one job |
| Able to specify a specific runner | Use runner of workflow calling action |

Reusable workflows provide a convenient means of reusing functionality without having to duplicate the code. But they do have one substantial drawback if you are looking to have them used consistently across repositories in your organization or enterprise. They cannot be enforced. You can’t require that they be used and run with each repository.

Fortunately, Actions includes a way to enforce that a workflow is always run for a given repository in certain use cases. That functionality is called *required* *workflows* and is the subject of the last section of this chapter.

# Required Workflows

As the name implies, *required workflows* allow admins to specify workflows that must run for a given set of repositories. This provides a way to mandate and enforce standards across a GitHub organization. When a workflow is required for a given repository, its execution becomes a required check that must be passed for a pull request to be processed and is clearly visible in the set of checks that are being run on proposed content changes.

# Beta State

As of the time of this writing, required workflows are still in beta.

It’s important to emphasize the *required* aspect of the workflow here. Repository administrators do not have the option to override them. So there are two key points anyone managing a repository in an organization that has required workflows should be aware of.

First, if there are pending pull requests already open in a repository when a workflow is designated as required, they will have these checks added and must also pass them. Since required workflows don’t run automatically, an additional push on the already-open pull request would be required to move things along.

Second, if a pull request is blocked by a required workflow and the situation cannot be correctly resolved via coding changes, the only option for proceeding is to ask the organization admin to remove the required workflow from the settings for all repositories.

Since I’ve highlighted the pull request scenario where a workflow can be required for completeness, next is some information on where they can’t be used.

## Constraints

The role in processing a pull request is actually more than an example; it’s the intended use case for required workflows at the time of this writing. `pull_request` and `pull_request_target` are the only valid trigger events where the `required` designation will cause the workflow to run. Related, there are a few scenarios where you will encounter an error trying to designate a workflow as required:

* If the YAML file is not valid syntax
* If the workflow does not have a valid trigger (*pull_request* or *pull_request_target*)
* If the file has already been selected as a required workflow in the organization
* If the required workflow references code-scanning actions

The last item deserves a bit more explanation. At the time of this writing, code-scanning actions are not allowed in required workflows. The reason is that code-scanning needs to be repository-specific, and it is configured via a different screen (see the [documentation](https://oreil.ly/iMMK8)). Currently, for example, if you attempt to configure a required workflow that uses the *codeql* actions, you’ll see a message like [Figure 12-4](#error-attempting-to-u).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1204.png)<br>
**Figure 12-4. Error attempting to use code-scanning in required workflow**

Required workflows can use secrets and variables—either from the target repository where they are being run or from the GitHub organization. As usual, repository secrets override organization secrets if they have the same name.

To finish the overview of required workflows and help complete your understanding, I’ll walk you through an example.

## Example

Anytime a pull request is initiated, contributors should be aware of the expected contribution standards for a repository. So, as a best practice, repositories that are open to pull requests should include a *CONTRIBUTING.md* file in the repository. (See the related [GitHub doc](https://oreil.ly/Lrt9b).)

Based on that convention, here is the listing for a simple workflow that checks for the presence of a *CONTRIBUTING.md* file and exits with failure if one is not found:

```yml
name: Verify existence of CONTRIBUTING.md file

on:
  push:
  pull_request:

jobs:
  verify:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - run: |
          [[ -f CONTRIBUTING.md ]] || ( echo "CONTRIBUTING.md file needs to be added to ${{ github.repository }} !" && exit 1 )
```

This should look fairly straightforward as far as the process. The single job simply checks out the source code and runs a bash command to see if a *CONTRIBUTING.md* file exists in the repository. If the file is found, the check will short-circuit and exit with success. If the file is not found, the second part of the check will be executed, then echo out a notification message and exit.

# Triggering Events

Notice that while this does have a `push` trigger option, as a required workflow, only the `pull_request` and `pull_request_tar⁠get` events will actually trigger this to run when it is required. Of course, the push trigger can still be useful for other use cases.

I’ll put this file in the *common* repository of the *rndrepos* organization (the same one that was used for the starter workflow previously). It will live there as *.github/workflows/verify-contrib-file.yml*.

To impose this as a required workflow on all repositories in the organization, I’ll go to the organization’s settings (the settings for the top-level organization, not any particular repository), then select the *Actions* menu on the side, followed by *General.*

Scrolling down to the bottom of the page is a place to add required workflows, as shown in [Figure 12-5](#adding-a-new-required).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1205.png)<br>
**Figure 12-5. Adding a new required workflow**

From here, I click the Add workflow button and select the repository with the required workflow (*common*) and enter the path to the workflow file ([Figure 12-6](#adding-a-specific-req)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1206.png)<br>
**Figure 12-6. Adding a specific required workflow**

By default, the required workflow will be active for all repositories. If you prefer, you can instead click the All repositories button and then select the *Select repositories* option in the list ([Figure 12-7](#choosing-the-option-t)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1207.png)<br>
**Figure 12-7. Choosing the option to apply the required workflow to selected repositories**

Clicking the gear icon will bring up a dialog with available repositories in the organization that can have the required workflow applied ([Figure 12-8](#option-screen-to-sele)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1208.png)<br>
**Figure 12-8. Option screen to select specific repositories to apply required workflow to**

After you are done with your selections, you can click the Apply selection button to save your choices. Whether you select All repositories or Selected repositories, be sure to click the Add workflow button at the bottom of the page to save your choices. Once you complete the process, you’ll see your selections registered on the page. And to the right is an ellipses that you can click to update or remove them ([Figure 12-9](#saved-required-workfl)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1209.png)<br>
**Figure 12-9. Saved required workflow options**

With these pieces in place, it’s time to look at an execution of the required workflow.

## Execution

Once the required workflow is configured to execute for a repository, it takes effect immediately. This means that if there is a pull request in progress, the new required workflow will be added to the checks for it. And approval of the pull request will be held until the new required workflow is satisfied. An example of this is shown in [Figure 12-10](#required-workflow-add). For new changes, the required check in the destination repository should run automatically.

If I then go and add a *CONTRIBUTING.md* file to the pull request branch, the required workflow check will run again and pass this time ([Figure 12-11](#passing-required-work)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1210.png)<br>
**Figure 12-10. Required workflow added to pull request in progress**

From this discussion and example, you see how required workflows can make the lives of admins easier by allowing them to define workflows that must run for repositories. They can be authored in any repository and shared (given the appropriate accesses) with other repositories. But it may be most useful to group them together in a common repository, as I did here.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1211.png)<br>
**Figure 12-11. Passing required workflow check as part of a pull request**

# Conclusion

In this chapter, we’ve covered a number of workflows patterns and related configurations that can make getting workflows and jobs set up and executed much easier.

Defining your own starter workflows makes it simple for others in your organization to get started with Actions and ensure newly created workflows are consistent and well structured.

Reusable workflows provide a way to reuse code and automation from other workflows. This supports a shared model for common use among many different users and repositories. Required workflows are similar to reusable workflows but can be mandated to be run and gate success for pull requests and pull request targets.

In the next chapter, I’ll continue the theme on advanced approaches to working in Actions by showing you some advanced techniques for executing scripts, interfacing with GitHub at a lower level, and orchestrating jobs within your workflows.

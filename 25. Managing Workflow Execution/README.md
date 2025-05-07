# Chapter 8. Managing Workflow Execution

By definition, GitHub Actions workflows are more declarative than imperative. This means that, instead of writing programming logic that defines how to get things done, you create workflows largely by declaring the triggers, jobs, steps, and runners that you want to use. And, for each step, you define which actions or commands are run to do the functionality. The actions themselves abstract out the programming.

However, just because you are writing workflows mostly by declaring elements in a YAML file, that doesn’t mean you can’t control more precisely their flow of execution. GitHub Actions provides a number of constructs and approaches for precisely managing how workflows are started and how they progress once started.

To finish out this section of the book, I’ll cover some of the key constructs and approaches for controlling how the execution of your workflow can be more precisely managed. Specifically, this chapter will cover these core areas:

* Advanced triggering from changes
* Triggering workflows without a change
* Dealing with concurrency
* Running a workflow with a matrix
* Workflow functions

# Advanced Triggering from Changes

I covered the basics of triggering your workflows in [Chapter 2](ch02.html#ch02). But, there may be situations where you need, or want, more advanced control over the triggering process. The idea is that triggers don’t have to be based just on general events. They can incorporate more specific criteria, including patterns for what’s changed and/or the type of activity that was happening when the event occurred.

For example, some of the core triggering events are based around GitHub objects, such as a GitHub issue. It’s simple to have that object be one of the triggers. Here’s a workflow for that:

```yml
on:
  issues:

jobs:

  notify-for-issue:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Something happened with an issue"
```

There’s the `on` keyword introducing the section of events that trigger running this workflow and then the `issues` trigger below that. It may look a bit strange to have a trigger with simply an ending colon and nothing after that, but it is valid syntax. The implication of this is that this workflow will be triggered for any and every kind of activity that occurs for an issue, such as creation, updating, or deletion.

If this is what you need, that’s great. But, if you need to refine more when your workflow runs, there are options you can supply for the triggers in the `on` section. These are referred to as *activity types*.

## Triggering Based on Activity Types

The *activity types* values allow you to specify what kinds of operations on the object will cause your workflow to run. For example, suppose I want this simple workflow to run only when I open a new issue.

I can consult [the GitHub documentation](https://oreil.ly/3P27b) to find the activity types for the particular item I’m triggering off of. [Figure 8-1](#activity-types-for-is) shows a section of this page for the issues trigger.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0801.png)<br>
**Figure 8-1. Activity types for issues (from GitHub documentation)**

Since this trigger supports an activity type of `opened`, I can add that to my workflow using the `types` keyword and standard YAML syntax, as shown in the next listing:

```yml
on:
  issues:
    types:
      - opened

jobs:

  notify-for-issue:
    runs-on: ubuntu-latest

    steps:
      - run: echo "An issue was opened"
```

You can also use YAML syntax to easily have the workflow triggered off of multiple activity types. For example, if you wanted the workflow to be triggered when an issue is opened, edited, or closed, you could use the following syntax:

```yml
on:
  issues:
    types: [opened, edited, closed]
```

Triggering from different kinds of activities is one way to get more precise about when your workflow runs. You can also trigger off of matching specific patterns for Git references and/or files in your repository. This is provided via defining filters within the *on* clause spec.

## Using Filters to Refine Triggers

Some triggering events allow using *filters* to further define when a workflow will run in response to the event. A filter is specified using a keyword that defines the type of entity to filter, and one or more strings that are specific names or patterns. The strings can use standard glob syntax (*, **, ?, !, +, etc.) to match multiples.

A good example is qualifying which branches and tags cause a workflow to run when a push event occurs. You can filter a list of branches and tags for the push event with wildcards for pattern matching:

```yml
on:
  push:
    branches:
      - main
      - 'rel/v*'
    tags:
      - v1.*
      - beta
```

You can also specify a set of branches to exclude via the keyword `branches-ignore`, tags to exclude via `tags-ignore`, or paths to exclude via `paths-ignore`*.* The use case for this is when it’s easier or more desirable to specify a set of branches or tags in your repository *not to* trigger off of rather than a set *to* trigger off of.

For example, you might define a workflow that does some kind of preproduction analysis on any work that’s in progress. And you might have that on all branches because a lot of branches need it for feature or bug development. But you don’t want to incur the overhead of running that workflow on any branches that are already in production or that have been approved as release candidates.

The trigger event specification for that use case might look like this (where production branches start with *prod* and release candidates are tagged with *rc**):

```yml
on:
  push:
    branches-ignore:
      - 'prod/*'
    tags-ignore:
      - 'rc*'
```

There are some related items for awareness around the use of patterns:

* These same include/exclude options apply to other events, such as *pull_request*.
* If you want to use special characters in your pattern that conflict with the glob patterns, you need to preface them with a backslash.
* Path filters are not evaluated for pushes of tags.
* The patterns for branches and tags are evaluated against *refs/heads* in the Git structure.
* You can’t include both the inclusive and exclusive keywords for the same event (for example, you can’t include both `branches` and `branches-ignore` for a push trigger).

The last point may lead to a question of how to filter on lower-level items that are contained within (inherit from) a broader set. The short answer is to leverage the `!`⁠glob pattern. For example, to filter out a set of beta releases that start with the same prefix as a larger set of releases, you could use this filter:

```yml
on:
  push:
    branches:
      - 'rel/**'
      - '!rel/**-beta
```

# ** Symbol

If you’re not familiar with the meaning of the ** symbol, in glob syntax, it matches filenames and directories recursively. Essentially, it matches on anything in a tree structure under the specified path.

Note that it does matter in which order you declare the patterns. If the pattern with the `!` character comes *afterwards*, it can be used as a refinement on the one before it. If the pattern with the `!` comes *before*, it would be overridden by the more inclusive pattern.

# Ensuring Your Workflow Exists for the Branch

Here’s a point to remember when you are adding a trigger that will target multiple branches (or a different one from where you are working). For operations like *push* that introduce branch-specific changes to the repository, the workflow file ultimately needs to exist on all branches where you want it to run.

Suppose, for example, that you are working on a pull request in a branch *other than main* for a workflow that needs to respond to a push on main when merged there. Until/unless you actually merge those changes into main, a push on main will not trigger it.

Likewise, if you want to do testing with it in the other branch, you’d need to ensure that branch was at least temporarily in the list for the push trigger.

If you are triggering off of a push or pull request event, you can refine those more to only trigger off of changes to particular file paths. The following code would cause the workflow to run anytime a change is pushed to a *.go* file:

```yml
on:
  push:
    paths:
      - '**.go'
```

Just as for branches and tags, there is the corresponding `paths-ignore` option. The following code will only run if there is at least one file changed outside of the data subdirectory at the root of the repository:

```yml
on:
  push:
    paths-ignore:
      - 'data/**'
```

Like the branches and tags options for triggering workflows, you can use the `!` character if you want to both include and exclude paths, since both `paths` and `paths-ignore` cannot be used together. Here’s an example using that syntax:

```yml
on:
  push:
    paths:
      - 'module1/**'
      - '!module1/data/**'
```

The workflow with this code will run if a file in *module1*, or one of its subdirectories, is changed, *unless* that file is in the *module1/data* subdirectory tree.

# Filter Pattern Cheat Sheet

GitHub provides a nice [cheat sheet for filter patterns](https://oreil.ly/lciBG).

In addition to being more precise about the kinds and patterns of change activity that can trigger a workflow, you can also trigger entire workflows to run in other ways besides something changing in the repository. This expands the utility of workflows to more manual use cases, as explained in the next section.

# Triggering Workflows Without a Change

GitHub Actions workflows can be triggered in ways other than the events already covered. There is a small set of triggers which are based on being called without a change happening in the repository. Examples include the `workflow_dispatch`, `repository_dispatch`, `workflow_call`, and `workflow_run` events.

The `_dispatch` events can be used if you need to trigger one or more workflows to run based on some activity that occurs outside of GitHub. For example, suppose I have a workflow in a repository called *create-failure-issue* that is used to create a GitHub issue with some data when a process fails. The workflow expects inputs of a title and a body text strings. And it requires a secret with a personal access token stored in a secret called *WORKFLOW_USE*. (More info on secrets is in [Chapter 9](ch09.html#ch09).)

I could invoke that workflow via a `curl` command as follows:

```sh
curl -X POST \
  -H "Authorization: Bearer ${{ secrets.WORKFLOW_USE }}" \
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

To illustrate the call and dispatch events for workflows, I put the previous code in its own workflow so I can invoke it in other ways. Here’s a listing with it in a workflow:

```yml
# This is a reusable workflow for creating an issue
name: create-failure-issue

# Controls when the workflow will run
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

This workflow includes both the `workflow_call` and `workflow_dispatch` events as triggers.

The `workflow_call` trigger allows this workflow to be used as a *reusable workflow*—one that can be called from other workflows. An example of a job that calls this workflow is shown in the next listing:

```yml
 create-issue-on-failure:

    needs: [test-run, count-args]
    if: always() && failure()
    uses: ./.github/workflows/create-failure-issue.yml
    with:
      title: "Automated workflow failure issue for commit ${{ github.sha }}"
      body: "This issue was automatically created by the GitHub Action workflow ** ${{ github.workflow }} **"
```

Reusable workflows can be used in place of the main code of a job via the `uses` statement. Reusable workflows are discussed in more detail in [Chapter 12](ch12.html#ch12).

The `workflow_dispatch` trigger allows you to run the workflow via the *Actions* tab, via the GitHub CLI, or via a REST API call. If your workflow has a `workflow_​dis⁠patch` trigger and if that workflow file is in the default branch, you’ll see a Run workflow button on the Actions tab when the workflow is selected. You can select that button and fill in any inputs you’ve defined. [Figure 8-2](#invoking-a-workflow-v) shows invoking the workflow through the Actions interface.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0802.png)<br>
**Figure 8-2. Invoking a workflow via a `workflow_dispatch` event**

While both `workflow_dispatch` and `repository_dispatch` can be invoked in some similar ways, the difference is that `workflow_dispatch` is intended for triggering a *specific* workflow, while `repository_dispatch` is intended for invoking *multiple* workflows within a repository. The latter is generally in response to some custom or external (to GitHub) event. An example could be an external CI process that needs to run multiple workflows in the repository to drive CD when a change occurs.

Finally, the `workflow_run` event trigger allows you to trigger the run of one workflow based on a separate workflow executing. The workflow in the following example will be triggered when another workflow with the name Pipeline runs to completion. In this case, it must also be on a branch starting with *rel* unless the *rel* branch name ends with *preprod*:

```yml
on:
  workflow_run:
    workflows: ["Pipeline"]
    types: [completed]
    branches:
      - 'rel/**'
      - '!rel/**-preprod'
```

The `completed` status here means that the pipeline ran to completion, which could be either success or failure. Alternatively, you can also use a status of `requested`, which implies only that the other workflow has been triggered*.* In that use case, this workflow would run at effectively the same time as the other one.

Note that there is also a `requested` status for the `workflow_run` event. That allows you to sequence execution of workflows in a similar manner, as the `needs` keyword allows you to sequence execution of jobs within a workflow.

Speaking of sequencing, this is a good opportunity to discuss how you can manage potential multiple instances of the same workflow running concurrently.

# Dealing with Concurrency

Usually, you will want (or need) to ensure that only a single instance of a workflow is running at a time. To accomplish that, the workflow syntax provides the `concurrency` keyword. This can be specified at the level of a job or an entire workflow.

To specify that you want only one instance of a job, or the entire workflow, to be allowed to execute, you specify a *concurrency group* as part of the concurrency clause. The concurrency group can be any string or expression. It can be supplied as the default argument to the `concurrency` keyword:

```yml
concurrency: release-build
```

If you add the concurrency clause to a job or workflow and if another instance with the concurrency clause is in progress, the new one will be marked as *pending*. If there was a previously pending instance with the same concurrency group, it will be cancelled, as shown in [Figure 8-3](#cancelled-job-in-work).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0803.png)<br>
**Figure 8-3. Cancelled job in workflow**

Note that if this is in the context of a job, any jobs dependent on the job that was cancelled will not be run. But other jobs in the same workflow that are not dependent on the cancelled job can still run.

If you want to have a more precise concurrency group, you can leverage elements of the *github context* as part of the expression, for example:

```yml
concurrency: {{ $github.ref }}
```

If there is already a running instance when your new instance gets triggered and you would prefer it to be cancelled, instead of having your new instance wait, you can specify *cancel-in-progress: true* as part of the concurrency clause, for example:

```yml
jobs:
  build:

    runs-on: ubuntu-latest

    concurrency:
      group: ${{ github.ref }}
      cancel-in-progress: true
```

To ensure that your concurrency group is unique to your workflow (to avoid cancelling other workflows in the same repository unintentionally), you can add the workflow property from the github context to it, as follows:

```yml
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true
```

# Using Undefined Context Values in Concurrency Groups

Be aware that since some context values are only defined for certain types of events, if you use them in concurrency group specifications and the triggering event does not provide that value, you will end up with a syntax error. For example, assume I have the following code:

```yml
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

concurrency:
  group: ${{ github.head_ref }}
  cancel-in-progress: true
```

Since the `head_ref` property of the *github* context is only defined when a pull request is done, if I did a push with this code, I would get a syntax error because the `head_ref` property is undefined on a push.

Here’s an example of such an error:

```yml
The workflow is not valid. .github/workflows/simple-pipe.yml (Line: 22,
Col: 14): Unexpected value ''
```

To prevent this, you can use a logical OR operation to have a fallback:

```yml
concurrency:
  group: ${{ github.head_ref || github.ref }}
  cancel-in-progress: true
```

Concurrency is one strategy to control how your workflow executes. At the other end from being concerned about multiple instances running is wanting to have multiple instances spun up, but for different combinations of data values. You can accomplish that using another strategy for controlling how your workflow executes: the *matrix strategy*.

# Running a Workflow with a Matrix

Sometimes, you may need to execute the same workflow multiple times based on different dimensions of data. For example, perhaps you need to run the same test cases across multiple browsers. Or you need to run the test cases across multiple browsers on each of multiple operating systems. For these kinds of cases, you can leverage the *matrix strategy* within GitHub Actions. You specify this strategy for the jobs in your workflow and define a matrix of dimensions you want to execute across. GitHub Actions will then generate jobs for each combination and execute them accordingly. The next listing shows an example of specifying the matrix strategy in a workflow:

```yml
name: Create demo issue 3

on:
  push:

jobs:
  create-new-issue:
    strategy:
      matrix:
        prod: [prod1, prod2]
        level: [dev, stage, prod]
    uses: rndrepos/common/.github/workflows/create-issue.yml@v1
    secrets: inherit
    with:
      title: "${{ matrix.prod }} issue"
      body: "Update for ${{ matrix.level }}"

  report-issue-number:
    runs-on: ubuntu-latest
    needs: create-new-issue
    steps:
      - run: echo ${{ needs.create-new-issue.outputs.issue-num }}
```

In this example, the processing will run across two products and three development levels. For each combination of product and level, it will call a *reusable workflow* to create a new GitHub issue. Ultimately, six instances of the job will run, and six new issues will be created. The return value for this is the last non-empty value returned by the processing.

[Chapter 12](ch12.html#ch12) goes into more details on options associated with using the matrix strategy, as well as more about reusable workflows.

# Continue on Error

The *continue-on-error* setting for jobs and steps can be used with the matrix strategy to allow the matrix processing to continue iterating through the combinations defined for your matrix. When this is specified, if one of the combinations fails, this will allow the workflow to continue processing the rest of the matrix.

The last part of this chapter discusses various functions you can use in your workflows to do simple processing that’s not complex enough to require an action but also not easy to do with a call to an external command. These are not declarative in the same sense as other parts of the workflow. But they are useful for convenience and for being able to alter the processing path if needed. These functions fall broadly into two categories: *inspection/formatting/transforming values* and *status/conditional checks*.

# Workflow Functions

There are a number of functions built in for use in workflows.  Some of them provide convenient ways to inspect, format, or transform strings or other values. [Table 8-1](#summary-available-functions) provides a brief summary, but more details can be found in the [GitHub Actions docs](https://oreil.ly/XDURW).

Table 8-1. Summary of available functions

| Function | Purpose | Usage |
| --- | --- | --- |
| `contains` | Checks if item is contained in a string or array. Return *true* if found. | `contains( search, item )` |
| `startsWith` | Checks if a string starts with a particular value. | `startsWith( searchString, searchValue )` |
| `endsWith` | Checks if a string ends with a particular value. | `endsWith( searchString, searchValue )` |
| `format` | Within a given string, replaces occurrences of {0}, {1}, {2}, etc. with the replacement values in the given order. | `format( string, replace​Value0, replaceValue1, ..., replaceValueN)` |
| `join` | Concatenates values in the array together into a string; uses comma as the default separator, but a different separator can be specified. | `join( array, optionalSeparator )` |
| `toJSON` | Pretty prints the specified value in JSON format. | `toJSON(value)` |
| `fromJSON` | Returns a JSON object or JSON datatype from the given value; useful to convert `env` variables from a string to another data type (such as boolean or integer) if needed. | `fromJSON(value)` |
| `hashFiles` | Returns a hash for the set of files that match the path specified. | `hashFiles(path)` |

These functions provide a lot of utility—some you can probably think of right away, and some are less obvious. For example, the `hashFiles` function can be used to create a unique hash to decide if using a previous cache is appropriate. (Caching is covered more in [Chapter 7](ch07.html#ch07).) Another common use case is using the `toJSON` function to pretty-print the contents of a context.

While it can be used to print a JSON representation of any value passed in, the `to⁠J⁠S⁠O⁠N` function is especially useful for dumping out large sets of data, which is what most contexts are. Here’s an example to print the contents of the *github* and *steps* contexts to the log:

```yml
jobs:
  print_to_log:
    runs-on: ubuntu-latest
    steps:
      - name: Dump GitHub context
        id: github_context_step
        run: echo '${{ toJSON(github) }}'
      - name: Dump steps context
        run: echo '${{ toJSON(steps) }}'
```

# Exposing Sensitive Data from Contexts

Keep in mind that values from secrets will be masked in the log (replaced with asterisks). But, it is still possible that sensitive data could be exposed via writing it to the log.

Beyond these data processing functions, there are also status functions to return the success/failure state of your workflow’s processing. These can be combined with conditional checks to determine if special handling needs to be done. You can use these to account for errors or to alter the execution of your workflow.

## Conditionals and Status Functions

You can use an `if` clause at the start of a job or a step to check a condition and determine if execution should occur or not. This can be done in a couple of ways.

You can check if context values are related to specific values. For example, the following code checks to see if the event is occurring on the main branch before allowing the job to execute. It also checks the value of the `os` property for the runner and reports information as part of a step:

```yml
name: Example workflow

on:
  push:

jobs:

  report:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/test'
    steps:
      - name: check-os
        if: runner.os != 'Windows'
        run: echo "The runner's operating system is $RUNNER_OS."
```

Also available are a set of *status functions* that can be used with conditionals to determine whether to execute a job or a step. [Table 8-2](#summary-of-status-che) shows the status functions along with explanations and example usage.

Table 8-2. Summary of status check functions

| Function | Meaning |
| --- | --- |
| `success()` | Returns true when none of the previous steps have been failed or cancelled |
| `always()` | Returns true and always proceeds even if the workflow has been cancelled |
| `cancelled()` | Returns true if the workflow was cancelled |
| `failure()` | When used with steps, returns true if a previous step failed; when used with jobs, returns true if a previous ancestor job (one that was in the dependency path) failed |

The syntax for these is fairly straightforward. You can write them as `if: ${{ success() }}`, but you can also use the simpler form of `if: success()`. And you can combine them with logical operators. An example of this is shown here:

```yml
create-issue-on-failure:

    permissions:
      issues: write
    needs: [test-run, count-args]
    if: always() && failure()
    uses: ./.github/workflows/create-failure-issue.yml
```

In this case, I am always checking for a failure, so I can create a GitHub issue to document the failure condition.

# Don’t Always Rely on Always()

In situations where you want to run a job or step regardless of success or failure *and* the potential for a critical failure may occur, it is best not to rely on `always()`. This is because you may end up waiting for a timeout to occur. The recommended approach for this situation is to use `if: success() || failure()` instead.

Finally, there is also a `timeout-minutes` setting that can be used to specify the maximum number of minutes that a job should be allowed to run before cancelling it. The default is 360.

# GITHUB_TOKEN Expiration

The GITHUB_TOKEN (discussed in [Chapter 6](ch06.html#ch06)) has a maximum lifetime of 24 hours. If the timeout is more than 24 hours, the token may be the deciding timeout.

# Conclusion

In this chapter, I’ve covered some more options for starting and managing the path of execution in your workflows.

The options for triggering your workflows are varied and extensive. As well as the wide assortment of events you can trigger on, you can also trigger on activity types and filters (aka patterns) for branches, tags, and files. Activity types give you finer-grained control over the dimension of *when* to trigger workflows that involve certain types of GitHub objects. Filtering gives you finer-grained control of *what* patterns of changes trigger the workflow.

A set of nonevent triggers is also available to start your workflow(s). These require manual starts or being called or triggered from other workflows or external events.

To ensure you can prevent multiple instances of workflows from running at the same time, GitHub Actions provides concurrency control. This is done via adding a concurrency clause with a group name that prevents other instances with the exact same group name from executing concurrently.

On the other hand, if you need to have multiple instances of your workflow running for different situations at the same time (such as testing across multiple browsers, platforms, etc.), you can add a matrix strategy to your workflow to automatically spin up instances across each of the dimensions.

In addition to the ways that instances of the workflows can be started and run, there are also workflow functions available to assist in manipulating data, checking success/failure, and altering the execution path during runs. These functions can assist with inspection, transformation, and output of strings and other types of objects, such as contexts.

Another set of functions can check the success/failure of parts of the workflow. These can be used with simple conditional logic to send execution down a different path or initiate automatic processing, such as generating failure reports.

This chapter concludes the section of the book on the basic building blocks for building productive workflows with GitHub Actions. In the next section of the book, I’ll spend some time helping you understand key issues you need to be aware of when your workflows are running—security and monitoring.

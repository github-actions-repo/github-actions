# Chapter 2. How Does Actions Work?

In [Chapter 1](ch01.html#ch01), you got acquainted at a high level with the overall framework and value of GitHub Actions. In this chapter, we’ll dive into the parts that make up GitHub Actions and how they work together, meaning what kicks them off, what happens when they run, and so on.

As a reminder, in the world of GitHub Actions, *actions* can refer to the following:

* The entire system for executing automated workflows in response to events
* The actual units of code and related pieces that implement individual actions

Following the convention suggested by GitHub, the book will use *GitHub Actions* or *Actions* (with an uppercase “A”) to refer to the system and *actions* (with a lowercase “a”) to refer to the code units.

To better understand the Actions environment, I’ll provide you with an overview of how the overall flow works. This includes the types of events that can start the automation and a high-level overview of the components that are involved in the execution of the automation. Throughout the chapter, I’ll offer some simple example code. This will give you a solid understanding of how the flow works.

# An Overview

At a high level, the GitHub Actions flow is this:

1. Some triggering *event* happens in a GitHub repository. This event is most often associated with a unique SHA1 (Secure Hashing Algorithm 1) value and a Git reference that resolves to an SHA1 value (a *ref*), such as a branch. But it may also be an event in GitHub that is *not* an update to a ref. An example would be a comment made in a pull request or an issue being updated.

2. A dedicated directory in the repository (*.github/workflows*) is searched for *workflow* files that are coded to respond to the event type. Many events can also include additional qualifiers. For example, a workflow can be set up to be triggered only when a *push* operation happens on the branch named *main*.

3. Corresponding workflows are identified, and new runs of the matching workflows are triggered.

The workflow object is the key piece here. A GitHub Actions workflow is a set of code that defines a sequence and set of steps to execute, similar to a script or a program. The file itself must be coded in [YAML](https://oreil.ly/RcYGd) format and stored in the *<repository>/.github/workflows* directory.

Workflow files have a [specific syntax](https://oreil.ly/7DAcu). A workflow contains one or more *jobs*. Each job can be as simple or as complex as needed. Once a workflow is kicked off, the jobs begin executing. By default, they run in parallel.

Jobs, in turn, are made up of *steps*. A step either runs a shell command or invokes a predefined GitHub action.  All of the steps in a job are executed on a *runner*. The runner is a server (virtual or physical) or a container that has been set up to understand how to interact with GitHub Actions.

I’ll go into more detail on each of these items later, but [Figure 2-1](#relationship-of-compo) illustrates the basic design.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0201.png)<br>
**Figure 2-1. Relationship of GitHub Actions components**

If this seems familiar, it’s probably because it’s a CI pattern. Some change is made that is automatically detected and that triggers a set of automated processes to run and respond to the change.

In GitHub Actions, the changes that signal that work needs to be kicked off are triggering *events* (aka *triggers*).

# Triggering Workflows

Events trigger workflows. They serve as the signal for work to start happening if a GitHub Actions workflow is present and the triggering event matches the start conditions specified in the workflow. An event can be defined in several different ways:

* A person or a process does some operation in a GitHub repository.
* A matching external trigger happens—that is, an event from outside of GitHub.
* A schedule is set up to run a workflow at particular times or intervals.
* A workflow is initiated manually, without an operation having to be done first.

I’ll dive into these different types more in [Chapter 3](ch03.html#ch03) and extensively in [Chapter 8](ch08.html#ch08), but an event triggered from an operation in a GitHub repository is probably the most common type. An example of this kind of event is a GitHub pull request. If you or a process initiates a pull request, then a corresponding *pull request event* is triggered. There is also a *push event* for code pushes. In GitHub, there are a large number of common operations you can do that can serve as triggers for a workflow.

There are also multiple ways to govern when workflows react to the triggers. To understand this, here’s the first piece of workflow syntax to become familiar with—the *on* clause. The *on* keyword and the lines that follow it define which types of triggers the workflow will match on and start executing. Some basic types of triggers and simple examples of the syntax for each follow:

The workflow can respond to a single event such as when a push happens:

```yml
on: push
```

The workflow can respond to a list (multiple events):

```yml
on: [push, pull_request]
```

The workflow can respond to event types with qualifiers, such as branches, tags, or file paths:

```yml
on:
  push:
    branches:
      - main
      - 'rel/v*'
    tags:
      - v1.*
      - beta
    paths:
      - '**.ts'
```

The workflow can execute on a specific schedule or interval (using standard cron syntax):

```yml
on:
  scheduled:
    - cron: '30 5,15 * * *'
```

# @interval Syntax

Syntax like @daily, @hourly, and so on is not supported.

The workflow can respond to specific manual events (more about these later):

```yml
on: [workflow-dispatch, repository-dispatch]
```

The workflow can be called from other workflows (referred to as a *reuse event*):

```yml
on: workflow_call
```

The workflow can respond to *webhook events*—that is, when a webhook executes and sends a payload. (See [the related documentation](https://oreil.ly/ox-qF) for more details on events and payloads in webhooks.)

The workflow can respond to common activities on GitHub items, such as adding a comment to a GitHub issue:

```yml
on: issue_comment
```

# Events That Trigger Only If the Workflow File Exists on the Default Branch

Be aware that a subset of less-common events will only trigger a workflow run if the workflow file (the YAML file in *.github/workflows*) is on the default branch (usually *main*). For those events, if you have the workflow file only on a non-default branch and you trigger the activity that would normally cause the workflow to run, nothing will happen.

You can trigger the event from another branch. But, for these special cases, the workflow file has to exist on the default branch, regardless of which branch the trigger actually happens on. This can present tricky situations when you are trying to develop a workflow in a different branch and prove it prior to doing a pull request.

To see if an event is one that can only be triggered for the default branch in your repository, go to [the documentation](https://oreil.ly/5xjgK) and check for a *Note* section that says, “This event will only trigger a workflow run if the workflow file is on the default branch.”

Deciding how you want your workflows to be triggered is one of the first steps to implementing functionality with GitHub Actions. To complete the picture, you need to understand more about the other parts that make up a workflow. I briefly touched on these in [Chapter 1](ch01.html#ch01), but I’ll explain more about them in the next section.

# Components

I’m using *components* here as an umbrella term (not an official one), for the major pieces that GitHub Actions defines for you to use to build and execute a workflow. For simplicity, I’ll just do a brief survey of each one to help you understand them from a higher level.

## Steps

*Steps* are the basic unit of execution you deal with when working with GitHub Actions. They consist of either invocations of a predefined action or a shell command to be run on the runner. Any shell commands are executed via a `run` clause. And any predefined actions are pulled in via a `uses` clause. The `steps` keyword indicates the start of a series of steps to be run sequentially.

The code listing that follows shows an example of three basic steps from a workflow. These steps check out a set of code, set up a *go* environment based on a particular version, and run the go process on a source file. In the YAML syntax, the `-` character indicates where a step starts. The `uses` clause indicates that this step invokes a predefined action. The `with` clause is used to specify arguments/parameters to pass to the action. And the `run` clause indicates a command to be run in the shell.

Note that steps can have a `name` associated with them as well:

```yml
steps:
- uses: actions/checkout@v4
- name: setup Go version
  uses: actions/setup-go@v2
  with:
    go-version: '1.14.0'
- run: go run helloworld.go
```

## Runners

*Runners* are the physical or virtual computers or containers where the code for a workflow is executed. They can be systems provided and hosted by GitHub (and run within their control), or they can be instances you set up, host, and control. In either case, the systems are configured to understand how to interact with the GitHub Actions framework. This mean they can interact with GitHub to access workflows and predefined actions, execute steps, and report outcomes.

In a workflow file, runners are defined for jobs simply via the `runs-on` clause. (Runners are discussed in more detail in [Chapter 5](ch05.html#ch05).)

```yml
runs-on: ubuntu-latest
```

## Jobs

*Jobs* aggregate steps and define which runner to execute them on. An individual job is usually targeted towards accomplishing one particular goal of the overall workflow. An example could be a workflow that implements a CI/CD pipeline with separate jobs for building, testing, and packaging.

Aside from the definition of the runner, a job in a workflow is like a function or procedure in a programming language. It is made up of a series of individual commands to run and/or predefined actions to call. This is similar to how a function or procedure in a programming language is made of individual lines of code and/or calls to other functions or procedures.

The outcome of the job is surfaced in the GitHub Actions interfaces. Success or failure is displayed at the level of the job, not the individual steps. It’s helpful to keep this success/failure status at the job level in mind when determining how much work you want any individual job to do. It’s also helpful when considering how much detail you want to know about success or failure within the workflow execution without having to drill down.

If you need more granular reports of success or failure displayed at the top level, you may want to put fewer steps in a job. Or, if you need less-granular indications of whether a set of steps succeeded or failed, you might put more steps into a job.

Building on the steps and runners previously shown, the next listing shows a simple job that does the checkout and setup and performs a build:

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: setup Go version'
        uses: actions/setup-go@v2
        with:
          go-version: '1.14.0'
      - run: go run helloworld.go
```

## Workflow

A *workflow* is like a pipeline. At a high level, it first defines the types of inputs (events) that it will respond to and under what conditions it will respond to them. This is what we talked about in the earlier section on events. The response, if the events and conditions match, is to then execute the series of jobs in the workflow, which, in turn, execute the steps for each job.

The overall flow is like a continuous integration process in that it responds to a particular kind of change and kicks off an automated sequence of work. The next listing shows an example of a simple workflow for processing Go programs built on the previous definitions:

```yml
1. name: Simple Go Build
2.
3. on:
4.   push:
5.     branches:
6.       - main
7.
8. jobs:
9.   build:
10.    runs-on: ubuntu-latest
11.    steps:
12.      - uses: actions/checkout@v4
13.      - name: Setup Go version
14.        uses: actions/setup-go@v2
15.        with:
16.          go-version: '1.15.1'
17.      - run: go run hello-world.go
```

Note that this workflow is written in YAML format. I’ll break down what’s happening in this file, line by line:

Line 1: The workflow file is assigned a name.

Line 3: This is the *on* identifier discussed in the section on events.

Lines 4–6: This workflow is triggered when a push operation is done to the branch *main* in this GitHub repository.

Line 8: This starts the jobs portion of the workflow.

Line 9: There is one job in this workflow, named *build*.

Line 10: This job will be executed on a runner system, hosted by GitHub, provisioned with a standard ubuntu operating system image.

Line 11: This starts the series of steps for this job.

Line 12: The first step is done via pulling in a predefined action. Note the way this is referenced. *actions/checkout@v4* refers to the relative path after github.com, so this really says it is going to run/use the action defined at [*github.com/actions/checkout*](https://github.com/actions/checkout).

Also notice that this is the only line in this step—no parameters need to be passed to this action because it assumes it is checking out the source from this repository since it is in this repository.

Line 13: The hyphen at the start of this line indicates this is the start of a second step. This line is giving the new step a name.

Line 14: The same step is pulling in another predefined action to set up the Go environment.

Lines 15–16: The *setup-go* action needs a parameter—the version of Go to use. The parameter is passed as an input to the action via a `with` clause.

Line 17: This is another step, one that simply runs a command as indicated by the `run` keyword. The command is to execute the `go run` command on an example file in the repository.

As a reminder, in order to be found, matched to event conditions, and executed automatically, this code needs to be stored in a YAML file in a special directory in a GitHub repository: *.github/workflows.* As an example, you could save the preceding code in *<your repository>/.github/workflows/simple-go-build.yml*. (Note the file extension here needs to be either .*yml* or *.yaml*, denoting YAML structure and syntax.)

# Workflow Execution

If you push the *.github/workflows/simple-go-build.yml* file and a corresponding *hello-world.go* file to a GitHub repository, you can see the workflow actually run right away. This is because the event condition set in the workflow (on a push to main) would match. So the workflow would be triggered and executed as soon as it is pushed.

GitHub repositories contain an *Actions* selection at the top of the project page. Selecting this puts you into a graphical interface where you can see runs of workflows and jobs. After pushing the workflow file, if you select the *Actions* tab at the top, you will see the execution of your simple workflow, as shown in the example in [Figure 2-2](#workflow-run).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0202.png)<br>
**Figure 2-2. Workflow run**

From here, you can select a run of a workflow and see the jobs that ran as part of the workflow and their statuses. [Figure 2-3](#run-at-the-job-level) shows the execution of the job from our simple workflow.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0203.png)<br>
**Figure 2-3. Run at the job level**

In later chapters, you’ll see how to use this interface to drill in further to what occurs when the various steps are executed and how to debug problems, when actions run, using the interface.

# Conclusion

*Actions* can refer to either the code that implements an action, the automated environment for defining and running those actions as part of a workflow, or both. In this chapter, I’ve focused on understanding the workflow, its components, and the way it is executed.

Workflows are like software pipelines. They can be initiated when a triggering event occurs (like continuous integration), and they aggregate one or more jobs to accomplish their overall task. Each job in turn aggregates one or more steps to do smaller units of work. The execution of the steps in a single job results in a success/failure outcome for the job, which feeds back into success/failure for the overall workflow.

Each job declares what kind of runner system (operating system and version) it will run in. And, at the lowest level, steps can invoke predefined GitHub Actions or run simple commands on that system.

Now that you have a basic understanding of how workflows work within GitHub Actions, [Chapter 3](ch03.html#ch03) will give you a similar understanding of how individual actions work.

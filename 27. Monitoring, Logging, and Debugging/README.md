# Chapter 10. Monitoring, Logging, and Debugging

At this point, you are hopefully comfortable with how to create, use, and manage GitHub Actions workflows, actions, and related pieces—when everything goes as planned. But what about those times when you need to quickly navigate through results, find more details, debug failures, or all of the above? No book on a new technology is complete without information on what to do, and where to look, when you need to dig deeper and/or things aren’t working. That’s the purpose of this chapter.

In this chapter, I’ll cover some of the built-in ways you can do the following:

* Gain more observability into what is happening with your workflows
* Work with previous runs of workflows
* Work with the framework’s debugging functionality to troubleshoot problems
* Customize log data and job summaries

Once you understand these techniques, you’ll be able to find the crucial data generated during the processing of your workflow and understand it at a deeper and more insightful level.

# Gaining More Observability

Observability can have a wide array of definitions. But the general goal of observability is always the same—to be able to quickly and easily identify and find the information you need about the current state of a process or system.

With GitHub Actions, there are a number of high-level ways to get that observability. At the most basic is the status output that is provided in GitHub through the integration with the *Actions* menu. While some of this has already been referenced in other parts of the book, there’s a more comprehensive view of status info available.

## Understanding Status at a High Level

As you’ve seen throughout the book, when workflows are triggered by events, GitHub Actions records information about that run, including the jobs that were executed, success/failure, duration, etc. You get to this list by clicking the *Actions* tab in the GitHub repository. [Figure 10-1](#portion-of-runs-for-a) shows a partial history of workflow runs for one of the author’s repositories.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1001.png)<br>
**Figure 10-1. Portion of runs for all workflows**

By default, this is showing you the runs for all workflows. (Note that *All workflows* is selected on the left in [Figure 10-1](#portion-of-runs-for-a).) In this view, in the list of runs on the right, you can look immediately under the commit message for a run to see which workflow it’s associated with. Those lines will also tell you which numbered run is for a given workflow and what kind of trigger initiated the run.

If you are interested in only seeing the runs for a single workflow, the simplest way to do that is to select the workflow in the list on the left of the screen. In [Figure 10-2](#single-workflow-selec), I’ve selected the one for *Simple Pipe* from the list. The list of workflow runs on the right has now changed to only show runs for that workflow.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1002.png)<br>
**Figure 10-2. Single workflow selected**

The list of workflow runs can also be filtered via options at the top of the list. One option is to filter via a search in the search bar at the top. Another option is to use the set of drop-down selectors at the top for Event/Status/Branch/Actor ([Figure 10-3](#selectively-filtering)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1003.png)<br>
**Figure 10-3. Selectively filtering list of runs by preset options**

# Filtering by Query

In the *Filter workflow runs* search box, you can use keywords with values to form simple search queries ([Figure 10-4](#example-categories-an)). This mechanism also provides some additional categories to search by, such as *workflow-name.*

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1004.png)<br>
**Figure 10-4. Example keywords and values to more precisely search**

For any run in the list, you can click the commit message associated with the workflow. This will open up the standard job graph, showing the ordering of execution of the different jobs in the workflow, as well as the success/failure status of each job.

As an alternative to having to go to the list of runs to see status, or if you need to see status on another location such as a web page, you can set up *badges* to quickly surface the status of a workflow running on a branch.

## Creating Status Badges for Workflows

GitHub Actions includes the ability to easily create a badge that always shows the latest status of one or more workflows. You can have multiple badges, and each can indicate the status for a combination of a branch and an event. An example of these badges is shown in [Figure 10-5](#badges-showing-status) in the lower left.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1005.png)<br>
**Figure 10-5. Badges showing status for two workflows for an action**

The badges here show status for two workflows, *Basic validation* and *e2e tests*. These are workflows defined for the repository backing this action. If you were to click the *e2e tests* title in the badge, you would be taken to the page for the latest runs of that workflow, as shown in [Figure 10-6](#result-of-clicking-on).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1006.png)<br>
**Figure 10-6. Result of clicking badge title**

To create one of these badges, you have a *Create status badge* option available in two different places. One option is on the main Actions screen when you have selected one specific workflow. A menu with three dots will be visible to the right of the search box (in the top right of the screen), and an option will be available there to create a badge ([Figure 10-7](#option-for-creating-a)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1007.png)<br>
**Figure 10-7. Option for creating a status badge on main Actions page**

Another place you can find this option is in the screen for an individual run of a selected workflow. You will see the same menu box with three dots in the upper right. When clicking it, you’ll have an option to create a status badge there ([Figure 10-8](#alternative-location)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1008.png)<br>
**Figure 10-8. Alternative location to create a status badge**

In reality, the status badge is a set of *Markdown* code that can be placed on any page that supports Markdown. The dialog that is brought up by clicking the option to create a status badge is simply a device to generate this code ([Figure 10-9](#dialog-to-generate-st)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1009.png)<br>
**Figure 10-9. Dialog to generate status badge code**

At the top of the dialog is an example of what the status badge would look like. This dialog is populated for the selected workflow, the default branch, and the default triggering event. But you have drop-down options to allow you to generate the code for other branches or events.

Once you have the selections the way you desire, you can just click the large green button at the bottom of the dialog labeled *Copy status badge code*. This will copy the generated code to your clipboard so that it can be pasted into a README file, a web page, or whatever location you intend. [Figure 10-10](#creating-a-readme-fil) shows an example of creating a *README.md* file with the status badge code pasted in at the bottom.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1010.png)<br>
**Figure 10-10. Creating a README file with a status badge**

[Figure 10-11](#readmemd-file-with-s) shows the resulting *README.md* file.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1011.png)<br>
**Figure 10-11. README.md file with status badge**

After this is in place, if a workflow run fails, the badge would change to reflect that state ([Figure 10-12](#status-badge-showing)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1012.png)<br>
**Figure 10-12. Status badge showing failing workflow**

# Creating Badges Manually

While the dialog makes creating badges simple, it is not necessary to use it. You can create your badges directly fairly easily. Here’s the syntax:

```yml
[![name](https://github.com/<repo-path>/workflows/
<.yml file name for workflow>/badge.svg?
branch=<branch name>&event=<event type>)]
(link to go to when badge is clicked)
```

And here’s an example URL:

```yml
[![Simple Pipe](https://github.com/gwstudent/greetings-add/
actions/workflows/pipeline.yml/badge.svg)](https://github.com/
gwstudent/greetings-add/actions/workflows/pipeline.yml)
```

You can find the Markdown syntax guide [in the documentation](https://oreil.ly/FDaqV).

Sometimes the simple status of the job is enough information to determine where to go next. Other times you may need to drill into the logs after selecting one of the jobs ([Figure 10-13](#drilling-into-logs-fr)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1013.png)<br>
**Figure 10-13. Drilling into logs from a job in a workflow run**

Being able to drill into any workflow run to see overall status, job information, and logs can provide most of the observability you will need for your day-to-day interaction with GitHub Actions. However, there may be times when you need to do more with past runs of a workflow. I’ll cover some ways to do that in the next section.

# Working with Past States

The Actions interface provides a continuous list of workflow runs that can be easily navigated through. Sometimes, though, simply looking at the info provided isn’t enough to answer a question about the run or determine how to solve an issue that occurred during the run.

Fortunately, you can also get to past states of the code base through the workflow run interface. And, if you are within 30 days of the time a run was completed, you can go back and re-run the workflow—as it was at the original point in time it was triggered.

## Mapping Workflow Versions to Runs

In the main list of workflow runs, you’ll find a set of three dots to the far right in each row. Clicking this set of dots provides the option to view the workflow file that was current at the point in time that run was done. If there was a pull request involved, you’ll also have a link to view that. And if you have permissions, a link to delete the run will also be available ([Figure 10-14](#options-for-viewing-t)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1014.png)<br>
**Figure 10-14. Options for viewing the workflow file**

Selecting the view workflow file option takes you to the version of the workflow file that was used in that run, so you can ensure you recall the set of code that was in place then ([Figure 10-15](#viewing-the-version-o)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1015.png)<br>
**Figure 10-15. Viewing the version of the workflow file that was used in a run**

From here, you also have a link under the *Workflow file for this run* line that will take you to the actual commit for that version (see section circled in [Figure 10-15](#viewing-the-version-o)). Clicking that link opens up the GitHub view of the changes for that commit. This can be very useful to understand/recall what changes were made at that point in time ([Figure 10-16](#viewing-the-commit-as)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1016.png)<br>
**Figure 10-16. Viewing the commit associated with the workflow file for a run**

Going back to the screen for the workflow run, you can select any of the jobs and see their run at that point in time on the runner.

Beyond getting to the past state of content, under certain conditions you can also re-run all or part of a recent workflow.

## Re-running Jobs in a Workflow

Sometimes it can be useful to be able to re-run all, or selected jobs, in a workflow. This can be done to remind yourself of the scenarios under which something succeeded or failed in the past. Or, you might do it to look at details from a run that you may not recall. Within a 30-day window from the initial run, GitHub Actions allows you to select a workflow, pick a specific run of that workflow, and re-run *all jobs*, *specific jobs*, or *all failed jobs*. It also gives you the ability to easily turn on debugging for that run, even if you didn’t have it turned on initially.

When you re-run jobs in a workflow, these are the key aspects to be aware of:

* You need to have write access to the repository to re-run jobs.
* The re-run will use the same commit SHA and Git ref from the original change that triggered the run.
* The re-run will use the privileges of the original *actor* that triggered the workflow, not the privileges of the actor that did the re-run.
* Re-runs are limited to a 30-day window from the initial run.
* Re-runs can’t be done once the retention limit for a log has expired.
* Re-runs of all failed jobs will include any dependent jobs, whether they failed or not.
* Debug logging is available for the re-run, but it must be selected.
* Since you are re-running jobs from a particular run, this does not result in a new workflow run being produced, even if you re-run all of the jobs.

In the next few sections, we’ll look at the various options for re-running jobs, starting with re-running all jobs.

### Re-running all jobs

Re-running all jobs in a workflow equates to running the entire workflow again. To get to this option, go to the *Actions* menu, select the workflow you’re interested in, and then select the specific run from the list.

Once you’ve selected a specific run, there will be a button with options to re-run the job in the upper-right part of the screen. If there were no failed jobs in that run, then you’ll have a button with an option that looks like [Figure 10-17](#option-to-re-run-all).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1017.png)<br>
**Figure 10-17. Option to re-run all jobs**

If there were failed jobs in the original run, then the button will have an additional option to re-run them. So to re-run all jobs in that case, you select an option from the drop-down list ([Figure 10-18](#re-running-all-jobs-w)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1018.png)<br>
**Figure 10-18. Re-running all jobs when there were failures**

After selecting the option, you’ll be presented with a confirmation dialog ([Figure 10-19](#confirmation-to-re-ru)). You also have an *Enable debug logging* option via the checkbox at the bottom. This is a nice feature as it allows you to see debug output for the new run even if you didn’t have that enabled for the original run. (I’ll discuss enabling debug output more generally later in this chapter.)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1019.png)<br>
**Figure 10-19. Confirmation to re-run all jobs**

# Re-running Jobs in a Pull Request

If you have a pull request, you can also get to the jobs screen for a workflow by selecting the *Checks* tab. Again, assuming you’re in the 30-day window, you can re-run all jobs, or individual jobs, via options accessible in the upper right of that screen. See [Figure 10-20](#re-run-option-in-pull) for an example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1020.png)<br>
**Figure 10-20. Re-run options in pull request Checks tab**

Failed jobs in the original run imply an additional option to re-run just them, as described in the next section.

### Re-running only failed jobs

If your previous run had jobs that failed and you want to re-run them again to gather data or review what happened, you can do that with the same approach as for re-running all jobs.

First, select the workflow run of interest, and then you’ll have the button at the top right with one of the options being to re-run the failed jobs ([Figure 10-21](#re-running-failed-job)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1021.png)<br>
**Figure 10-21. Re-running failed jobs from a run**

After that, you’ll be prompted with a confirmation dialog. As with the option to re-run all jobs, you can enable debug logging via the checkbox at the bottom of the dialog, as shown in [Figure 10-22](#confirmation-for-re-r).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1022.png)<br>
**Figure 10-22. Confirmation for re-running a failed job**

One key point to notice here is that re-running all failed jobs will automatically include running all of their dependent jobs as well.

There may be times when you only want to re-run a specific job. That functionality is provided, though getting to it is not as obvious.

### Re-running individual jobs

If you want to re-run an individual job, navigate to the individual run you’re interested in first. Then, instead of selecting the button to re-run jobs, just hover over the job name on the left side. Then you’ll see a set of circular arrows appear, as shown in [Figure 10-23](#option-to-re-run-indi).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1023.png)<br>
**Figure 10-23. Option to re-run individual job**

Alternatively, with the job selected and the log open for it, you will have an option to re-run the job from a button displaying the same circular arrows in the bar above the log ([Figure 10-24](#option-to-re-run-job)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1024.png)<br>
**Figure 10-24. Option to re-run job from log**

When re-running an individual job, any outputs, artifacts, or environment protection rules that are accessed will be provided from the previous run. Any environment protection rules that passed in the previous run will automatically pass in the re-run. (For more about environment protection rules, see the section on deployment environments in [Chapter 6](ch06.html#ch06).)

Also, like running failed jobs, any dependent jobs of the selected job will also be re-run. [Figure 10-25](#re-running-a-single-j) shows the set of jobs executed by choosing to re-run only the build job.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1025.png)<br>
**Figure 10-25. Re-running a job and its dependent jobs**

Any re-run will result in additional executions being logged and tracked for the specific run. After the additional runs are done, there will be a new control that can be used to view them.

### Viewing multiple run attempts

After you have completed another instance of the run of the workflow, you will have an additional control in the upper right of your screen allowing you to select among the different instances of the run ([Figure 10-26](#additional-control-fo)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1026.png)<br>
**Figure 10-26. Additional control for selecting instance of run to view**

Once you have multiple instances of the run, if you are viewing an older run (not the most recent attempt), Actions will place a banner at the top of the list to make sure you are aware ([Figure 10-27](#banner-identifying-th)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1027.png)<br>
**Figure 10-27. Banner identifying that you’re viewing an older attempt**

As noted in the preceding sections, when you re-run a workflow or jobs in a workflow, you can enable debug logging for the re-run. This is a useful shortcut for getting debug information on new attempts at previous runs. But debug logging is a much more extensive and powerful tool for troubleshooting in GitHub Actions. In the next section, we’ll discuss how to activate and use this functionality more widely with your workflows.

# Debugging Workflows

While GitHub Actions provides a useful level of logging by default, the default is not always enough to understand why a particular situation is occurring when your workflow is run. If the workflow logs don’t provide enough information to help troubleshoot and diagnose a problem, you can enable more extensive debug logging.

Debug logging can provide much more insight into what is happening during each step executing in a workflow. Here’s an example of logging, without debugging turned on, for a step that downloads an artifact:

```yml
Run actions/download-artifact@v4
  with:
    name: greetings-jar
Starting download for greetings-jar
Directory structure has been set up for the artifact
Total number of files that will be downloaded: 2
Artifact greetings-jar was downloaded to
/home/runner/work/greetings-add/greetings-add
Artifact download has finished successfully
```

And here’s the same code executing with debug logging enabled:

```log
##[debug]Evaluating condition for step: 'Download candidate
artifacts'
##[debug]Evaluating: success()
##[debug]Evaluating success:
##[debug]=> true
##[debug]Result: true
##[debug]Starting: Download candidate artifacts
##[debug]Loading inputs
##[debug]Loading env
Run actions/download-artifact@v4
##[debug]Resolved path is
/home/runner/work/greetings-add/greetings-add
Starting download for greetings-jar
##[debug]Artifact Url:
https://pipelines.actions.githubusercontent.com/
twiKCH6yMYWNpVZN5ufKzO6UmKFpiP8Eti5VHW94Bd8b6qXCg7/
_apis/pipelines/workflows/4081071917/
artifacts?api-version=6.0-preview
Directory structure has been set up for the artifact
##[debug]Download file concurrency is set to 2
Total number of files that will be downloaded: 2
##[debug]File: 1/2.
/home/runner/work/greetings-add/greetings-add/test-script.sh
took 39.239 milliseconds to finish downloading
##[debug]File: 2/2.
/home/runner/work/greetings-add/greetings-add/build/libs/
greetings-add-2023-02-03T04-26-49.jar took 39.734 milliseconds
 to finish downloading
Artifact greetings-jar was downloaded to
/home/runner/work/greetings-add/greetings-add
Artifact download has finished successfully
##[debug]Node Action run completed with exit code 0
##[debug]Set output download-path =
/home/runner/work/greetings-add/greetings-add
##[debug]Finishing: Download candidate artifacts
```

As you can see, there’s a lot more information available to you when you enable debugging. This includes the actual locations of files and directories on the runner, timings, additional return codes, etc. Arguably, there is also more *noise* to wade through. But when you really need to understand what’s happening to resolve an issue, the additional debug information can be invaluable.

There are two kinds of debug logging that GitHub provides. You can enable either or both.

## Step Debug Logging

When you turn on *step debug logging*, you get an increased level of detail around each job’s execution. You can think of it as GitHub Actions giving you a detailed breakdown of what it’s doing behind the scenes for each step. This information generally falls into a few categories:

* Prep or finish work to set up/clean up prior to a step’s main logic:

  ```log
  ##[debug]Starting: Download candidate artifacts
  ##[debug]Loading inputs
  ##[debug]Loading env
  ...
  ##[debug]Node Action run completed with exit code 0
  ##[debug]Set output download-path =
  /home/runner/work/greetings-add/greetings-add
  ##[debug]Finishing: Download candidate artifacts
  ```

* Information on settings:

  ```log
  ##[debug]Download file concurrency is set to 2
  ```

* Timings:

  ```log
  ##[debug]File: 1/2.
  /home/runner/work/greetings-add/greetings-add/test-script.sh
  took 39.239 milliseconds to finish downloading
  ##[debug]File: 2/2.
  /home/runner/work/greetings-add/greetings-add/build/libs/
  greetings-add-2023-02-03T04-26-49.jar took 39.734 milliseconds
  to finish downloading
  ```

* Full URLs on GitHub and fully resolved paths on the runner system:

  ```log
  ##[debug]Artifact Url:
  https://pipelines.actions.githubusercontent.com/
  twiKCH6yMYWNpVZN5ufKzO6UmKFpiP8Eti5VHW94Bd8b6qXCg7/
  _apis/pipelines/workflows/4081071917/
  artifacts?api-version=6.0-preview

  ##[debug]File: 1/2.
  /home/runner/work/greetings-add/greetings-add/test-script.sh
  ```

* Results of evaluating expressions, conditionals, etc.:

  ```log
  ##[debug]Evaluating condition for step: 'Download candidate
  artifacts'
  ##[debug]Evaluating: success()
  ##[debug]Evaluating success:
  ##[debug]=> true
  ##[debug]Result: true
  ```

One advantage of turning on debugging is that it produces a lot of additional information in the logs. However, this can also be a disadvantage when you are trying to parse through the information. In the browser-based log view through the Actions tab, the debug messages are highlighted so they stand out more ([Figure 10-28](#debug-messages-in-log)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1028.png)<br>
**Figure 10-28. Debug messages in logs**

Notice that at the top of the job log display, there is a box to *Search logs*. This is useful to have, but at the time of this writing, it appears to only find hits if the step in the display has already been expanded. You can expand the step and then do the search to see results.

One other approach for finding content easily in logs is to select the option to *View raw logs* from the settings ([Figure 10-29](#option-to-view-raw-lo)). This will display the plain-text version (of the job and step logs) in your browser window. You can then download those to your system or use standard find techniques such as Ctrl-F to search for text.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1029.png)<br>
**Figure 10-29. Option to view raw logs**

Another option for looking at logs outside of the Actions interface is to download them via the *Download log archive* menu selection ([Figure 10-30](#option-to-download-lo)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1030.png)<br>
**Figure 10-30. Option to download log archive**

As the name implies, when you select the option to download the log archive, a zip file with the collection of individual logs is downloaded to your system. After downloading, you can open up the zip file and view the logs inside ([Figure 10-31](#inside-the-log-archiv)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1031.png)<br>
**Figure 10-31. Inside the log archive**

The logs are organized by folders for the jobs with files for the individual steps. The step files are just standard text files that can be viewed and searched with whatever application you choose.

There may be times when you need to dive deeper than just the workflow to understand the cause of an issue. For those times, GitHub Actions provides additional diagnostic logging that can be activated.

## Debugging the Runner Environment

*Runner diagnostic logging* provides you with detailed information about what is happening on the actual runner system when your workflow is being executed.This includes data such as how the runner app is connecting and interacting with GitHub, as well as the low-level details on the data, transactions, and processes being executed for each job in your workflow. You typically won’t need this level of detail, but there are use cases where it can come in handy, such as these:

* Understanding a lower level of what an action you are using is doing
* Looking into timing issues
* Inspecting the data that is getting passed through processes
* Looking into connection or authentication issues—especially for self-hosted runners
* Understanding system/code interactions—especially on custom environments in self-hosted runners

To activate runner diagnostic logging, you must have debugging activated either via setting the repository secret/variable for *ACTIONS_STEP_DEBUG* or, if you are re-running a job, by selecting the *Enable debug logging* checkbox. (Both of these options to get debugging information are described in other sections of this chapter.)

Then you need to instantiate a secret or variable named *ACTIONS_​RUNNER​_​DEBUG* and set its value to *true*. (This is the same process you would use for instantiating *ACTIONS_STEP_DEBUG*, just with a different name for the secret/variable.) Once you do that, the additional logs will be generated, but since they are runner-wide and not related to a specific job, you won’t be able to see the results in the browser interface. Instead, they are included with the log archives that you can download. So you need to select the *Download log archive* option in the menu ([Figure 10-30](#option-to-download-lo)) when you are looking at a job log to get them.

[Figure 10-32](#runner-diagnostics-in) shows an example of the runner diagnostic logs in the downloaded log archive after expanding it.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1032.png)<br>
**Figure 10-32. Runner diagnostics in downloaded log archive**

Within the *runner-diagnostic-logs* directory, there is a separate zip file for each job that was part of the workflow run. Uncompressing these zip files results in a separate directory with two log files for the job. One log file is for the interaction of the runner system with GitHub (starting with *Runner_*), and the second log file is for the actual run of the steps in the job (starting with *Worker_*). Examples of an expanded set of runner diagnostic logs are shown in [Figure 10-33](#expanded-runner-diagn).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1033.png)<br>
**Figure 10-33. Expanded runner diagnostic logs folder**

# UnknownBuildNumber

In case you are wondering about the *UnknownBuildNumber* reference in the log names, it is simply a placeholder until such time as additional functionality is added to include the folder name when generating logs.

Here I’ll summarize the differences between the two types of debugging:

> *ACTIONS_STEPS_DEBUG*
>> * Causes the GitHub Actions engine to emit debugging information about steps
>> * Is unrelated in functionality to *ACTIONS_RUNNER_DEBUG*
>> * Results can be viewed through Actions interface job logs or downloaded as part of log archive
>> * Can be activated through a repository secret, a repository variable, or an option when selectively re-running jobs
>
> *ACTIONS_RUNNER_DEBUG*
>> * Causes the runner to upload diagnostic logs at the end of the job
>> * Is unrelated in functionality to *ACTIONS_STEP_DEBUG* but requires *ACTIONS​_​STEP​_​DEBUG* to be activated in order to produce the logs
>> * Results are only available via an additional directory in a downloaded log archive
>> * Can be activated through a repository secret or a repository variable

Per the last point in each of these two lists, to be able to generate the debugging information for steps and runner diagnostic information, you do first need to activate (switch on) the functionality in your repository. The next section details your options for doing that.

## Activating Debugging

To activate the debugging functionality for the steps or the runner system, you must define some items in a repository to turn the functionality on. There are two kinds of switches you can define for this—secrets or variables.

# Turning On Debugging for Re-runs

As noted in the section on re-running jobs in this chapter, even without having the secret or variable set, it is possible to select an option for a particular re-run to get the step debug information for that run.

As described in [Chapter 6](ch06.html#ch06), the process for defining a secret or a variable is very similar. To create the secrets or variable to enable debug information, you would enter *ACTIONS_STEP_DEBUG* as the name and *true* as the value. Then just add the secret or variable. [Figure 10-34](#adding-a-new-secret-t) shows the dialog when adding a secret to switch on debugging.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1034.png)<br>
**Figure 10-34. Adding a new secret to turn on step debugging**

Turning on the functionality to provide the runner diagnostic information is the same process, except the secret or variable needs to be named *ACTIONS_​RUN⁠NER​_​DEBUG*.

One other quick note on debugging related to GitHub Actions: if you find yourself running into issues trying to use self-hosted runners, the GitHub Actions documentation has a good [write-up](https://oreil.ly/sh4ur).

As opposed to getting more details from the system through debug info in logs, there may be times when you want to define and surface additional information, context, or structure into the logs. To do this, you can leverage the same functionality that GitHub Actions itself uses for this.

# Working with the System PATH

There can be times when it is useful to add additional directories to the path your runner system is using while executing a job. For example, you might want to reference a custom tool or reference some temporary generated content or data.

Since each job runs on its own runner, there is a consistent system PATH used for all steps and actions in the job. You can print this out via a simple `echo "$PATH"` in a step or action.

However, you also have the ability to prepend a directory onto the system PATH variable via `echo "{path}" >> $GITHUB_PATH`. When you do this, the updated PATH will be available for subsequent steps in the job (but not the one actually updating it).

# Augmenting and Customizing Logging

GitHub Actions is designed to provide well-formatted logging, job summaries, and related records that are easy to drill into. In a browser interface, this includes controls and options to navigate the collection of data. However, you are not limited to only using the output provided by Actions. You can augment and customize the logging and summaries that are generated by your workflows.

## Adding Your Own Messages in Logs

You can easily add your own *user-supplied* messages in logs. You have a number of types of messages to choose from, including `notice`, `warning`, `debug`, and `error`. The process itself is simple, using workflow commands that activate functionality by using the `echo ::<function>::` syntax as part of a step.

For example, you can echo out any message you want as a debug message by prefacing it with `::debug::`.

# Prerequisite for Displaying Custom Debug Messages

You will need to have the secret or variable set to turn on ACTIONS_STEP_DEBUG (or select the option to enable debug logging if doing a re-run) as described in the previous sections of this chapter—otherwise the output of your debug message will not show up.

An example is shown here:

```yml
echo "::debug::This is a debug message"
```

The other types of messages can be displayed with the same process, just substituting the appropriate type for *debug*, as shown in this listing:

```yml
echo "::warning::This is a warning message"
echo "::notice::This is a notice message"
echo "::error::This is an error message"
```

When you use the warning, notice, or error messages in a workflow, that will also produce annotations in the output for that workflow’s run. [Figure 10-35](#default-annotations-f) shows the kind of default annotations that are produced. (In this case, there is only one job in the workflow, and it is named *create_issue_on_failure*.)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1035.png)<br>
**Figure 10-35. Default annotations from failure, notice, and warning messages**

# Additional Parameters for Custom Messages

For error, warning, and notice messages, the documentation states that extra parameters can also be passed in to create additional annotations ([Table 10-1](#additional-parameters)). However, as of the time of this writing, the additional parameters don’t seem to work in all cases and don’t result in the expected annotations. [Table 10-1](#additional-parameters) lists the parameters and their meaning. They are being included here for completeness and in case this is fixed in a future update.

Here is an example of using these parameters in a custom message:

```yml
echo "::error file=pipe.yaml,line=5,col=4,endColumn=8::
Operation not allowed"
```

Table 10-1. Additional parameters and values for messages

| Parameter | Definition |
| --- | --- |
| file | Name of file |
| col | Starting column |
| endCol | Ending column |
| line | Starting line |
| endLine | Ending line |

Beyond messaging, there are additional formatting options available to you for logging.

## Additional Log Customizations

If you want to add additional display functionality to a log from your workflow runs, you can add custom code to your workflows to provide the same kind of special formatting. These include the following options.

### Grouping lines in a log

Using the `group` and `endgroup` workflow commands, you can group content in a log into an expandable section. The `group` command takes a `title` parameter, which is what will be shown when the expandable section is collapsed.

As an example, assume you have the following code:

```yml
    steps:
      - name: Group lines in log
        run: |
          echo "::group::Extended info"
          echo "Info line 1"
          echo "Info line 2"
          echo "Info line 3"
          echo "::endgroup::"
```

Then, when the workflow is executed, you’ll have a grouping in your log like the one in [Figure 10-36](#log-grouping).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1036.png)<br>
**Figure 10-36. Log grouping**

You can also hide sensitive information in logs through a process called *masking*.

### Masking values in logs

GitHub Actions automatically masks the values of secrets you have defined for your repository. But you can explicitly mask the value of any string or variable in a log to prevent its value from being printed in clear text in the log. This is done by using a workflow command to echo *::add-mask::* with the string or environment variable. Example code for masking a variable follows:

```yml
jobs:
  log_formatting:
    runs-on: ubuntu-latest

    env:
      USER_ID: "User 1234"

    steps:
      - run: echo "::add-mask::$USER_ID"

      - run: echo "USER_ID is $USER_ID"
```

With this code in place, when this job is run, the output will look like [Figure 10-37](#masked-variable-value).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1037.png)<br>
**Figure 10-37. Masked variable value**

Note line 6 in the second step where the USER_ID value is replaced by *****.

# Masking Configuration Variables

You can also mask configuration variables simply by using the *vars* context. For example, if you have a repository variable *USER_ID2* configured in your repository, the following code can be used to mask its value in the log:

```yml
   env:
      USER_ID: ${{ vars.USER_ID2 }}

    steps:
      - run: echo "::add-mask::$USER_ID"
      - run: echo "USER_ID is $USER_ID"
```

Configuration variables are discussed in more detail in [Chapter 6](ch06.html#ch06).

In addition to the logging customizations, you can create custom job summaries to be displayed as part of the output for your workflows.

## Creating a Customized Job Summary

Job summaries refer to the output displayed on the summary page of a workflow run. Their primary purpose is to gather and show content about the run so that you don’t need to drill into the logs to see key information. Some actions produce their own summaries automatically when you use them. An example of a generated summary is shown in [Figure 10-38](#example-of-a-generate).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1038.png)<br>
**Figure 10-38. Example of a generated summary from an action**

There can be a job summary for each job, but this is optional. A summary for a job is a grouping of any summaries for the individual steps in the job. To add summary information for a step to a job’s summary, you simply direct the summary information to the special environment variable *GITHUB_STEP_SUMMARY*. This is an environment variable that is set to the value of a temporary file containing the summary of the step that you want to format. The contents of that file are ultimately read and attached to the output. There is one of these per job.

The summaries use the format of [GitHub-flavored Markdown](https://oreil.ly/MBpFS), and you can use that syntax to display nicely formatted output. Here’s example code showing Markdown being added for different steps in multiple jobs:

```yml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: |
         echo "Do build phase 1..."
         echo "Build phase 1 done :star:" >> $GITHUB_STEP_SUMMARY

      - run: |
         echo "Do build phase 2 with input..."
         echo "Build phase 2 done with parameter ${{ github.event.inputs.param1 }} :exclamation:" >> $GITHUB_STEP_SUMMARY

  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Do testing..."

      - name: Add testing summary
        run: |
          echo "Testing summary follows:" >> $GITHUB_STEP_SUMMARY
          echo " | Test | Result | " >> $GITHUB_STEP_SUMMARY
          echo " | ----:| ------:| " >> $GITHUB_STEP_SUMMARY
          echo " |  1   | :white_check_mark: | " >> $GITHUB_STEP_SUMMARY
          echo " |  2   | :no_entry_sign: | " >> $GITHUB_STEP_SUMMARY
```

[Figure 10-39](#custom-output-summari) shows the summaries generated from this code in the run of the workflow.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_1039.png)<br>
**Figure 10-39. Custom output summaries**

# GitHub Markdown Emojis

For a complete list of the emojis available for you to use in GitHub Markdown, see [the Gist documentation](https://oreil.ly/PgckZ).

# Conclusion

GitHub Actions provides a number of built-in features to help you understand the execution of your workflow runs at multiple levels. This includes clear, easy-to-find status information in the browser interface through the list of workflow runs, job graphs, and selectable filters.

At a lower level, you can dig into logs and turn on debugging information for the steps in your workflow that will be visible in your logs.

At the deepest level, you can enable runner diagnostic logs to see exactly what is being executed on the runner system throughout a workflow run. These logs are generated separately and have to be downloaded to be accessed and viewed.

When you want to provide more diagnostic or information messages in your logging, Actions makes that easy to do with simple workflow commands that can be used to echo out different types of messages.

Finally, when your jobs are completed, you can provide customized job summaries with as much detail as needed/desired.

At this point, you’re familiar with the core functionality and use of actions and workflows and how to handle critical factors like security and debugging. You’re now well-positioned to move on to learning and implementing some advanced techniques. [Chapter 11](ch11.html#ch11) will start you on that path by showing you how to create your own custom actions.

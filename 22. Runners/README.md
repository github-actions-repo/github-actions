# Chapter 5. Runners

Regardless of what functionality you implement with GitHub Actions, there has to be a place to execute that functionality—a virtual or physical system with enough resources to process a job, and one that is configured to interact with the Actions control plane as it dispatches jobs. In Actions terminology, the systems where jobs in a workflow are executed are referred to as *runners*.

At a high level, you have two choices for the runner systems. You can use default systems provided by GitHub or you can configure, host, and use your own. I’ll explore both options in this chapter along with their key attributes, usage, and pros and cons. I’ll start by looking at the systems that GitHub automatically provides by default.

# GitHub-Hosted Runners

The runners provided by GitHub are the simplest and easiest way to execute jobs in workflows. Every GitHub-hosted runner is created as a new virtual machine (VM) with your choice of Ubuntu Linux, Windows Server, or macOS as the operating system. An advantage of using the GitHub-hosted runners is that GitHub takes care of the needed/required upgrades and maintenance for the VMs.

When executing workflows with these runners, no additional setup or configuration is required for each job beyond the simple runner declarations, such as `runs-on: ubuntu-latest`.

These labels in the workflow’s YAML cause GitHub to provision and start a virtual runner system with a particular operating system and environment for a given job. [Table 5-1](#mappings-of-runner-la) is taken from the GitHub Actions documentation and shows how the different labels map to the OS environments on the VMs provided by GitHub as runners (as of the point in time this is being written).

Table 5-1. Mappings of runner labels to OS environments

| Environment | YAML label | Included software |
| --- | --- | --- |
| Ubuntu 22.04 beta | `ubuntu-22.04` | [ubuntu-22.04](https://oreil.ly/WdVWs) |
| Ubuntu 20.04 | `ubuntu-latest` or `ubuntu-20.04` | [ubuntu-20.04](https://oreil.ly/u0VVb) |
| Ubuntu 18.04 | `ubuntu-18.04` | [ubuntu-18.04](https://oreil.ly/-WOxZ) |
| macOS 12 beta | `macos-12` | [macOS-12](https://oreil.ly/ExrZ1) |
| macOS 11 | `macos-latest` or `macos-11` | [macOS-11](https://oreil.ly/WHdgq) |
| macOS 10.15 | `macos-10.15` | [macOS-10.15](https://oreil.ly/oGsB-) |
| Windows Server 2022 | `windows-latest` or `windows-2022` | [windows-2022](https://oreil.ly/XMpVW) |
| Windows Server 2019 | `windows-2019` | [windows-2019](https://oreil.ly/RVFbw) |
| Windows Server 2016 | `windows-2016` | [windows-2016](https://oreil.ly/04fBj) |

# Costs for Using Non-Linux Systems

If you are using the macOS or Windows Servers GitHub-provided environments and are on a GitHub plan that you pay for, remember that they cost more to use per minute than the corresponding Ubuntu systems. The multiplier is 2x for Windows and 10x for macOS. The details on this were covered under the section on costs in [Chapter 1](ch01.html#ch01).

While we’ve used the *ubuntu-latest* label in the jobs so far in the book, note that there are similar *latest* options available for Windows (Server) and for macOS. You can also specify versions of an OS to use if there is a corresponding label available. You should be aware, though, that some version-specific labels refer to beta versions of an OS. So, unless you have a specific need to use beta features, it is recommended to use the “*-latest*” version label to get the most recent production versions of the OS.

# Support for Latest and Beta Images

In the context here, *latest* does not necessarily mean the most recent version from the vendor. Rather, it is the most recent stable version supported by GitHub.

Also, beta images may not be supported by GitHub. They are as is.

## What’s in the Runner Images?

If you want to understand details about the runner images used by GitHub, Actions makes that easy. Just go to the workflow logs and expand the *Set up job* section. A few lines down under that will be a *Runner Image* section. If you expand this one, you’ll see a link for *Included Software*. An example is shown in [Figure 5-1](#finding-the-included).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0501.png)<br>
**Figure 5-1. Finding the Included Software link**

Clicking the link will take you to a web page that lists all the software included in this environment. The corresponding page from the link is shown in [Figure 5-2](#readme-for-ubuntu-vir).

At a higher level in this same [project](https://oreil.ly/Xlxuj) are the *[linux](https://oreil.ly/YxpTJ)*, [*macos*](https://oreil.ly/3fwcO), or [*win*](https://oreil.ly/uyNdi) folders. Within these folders are the configuration files and scripts to set up the different runner images. Also in the same folders are *Readme.md* files for the different currently supported versions. Within those files, you can find a listing for the included software.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0502.png)<br>
**Figure 5-2. Readme for Ubuntu runner image**

# Viewing the SBOM for GitHub-Hosted Runners

A key aspect of security is visibility into the supply chain for infrastructure systems. This is typically managed by producing a software bill of materials (SBOM) to see what software was on the system during your workflow runs.

SBOMs can be provided for scanning to look for any vulnerabilities as well as to have a complete point-in-time list. SBOMs can also be packaged up and delivered with artifacts.

For GitHub-hosted runners, SBOMs are available for the runner images. They can be found in [the release assets for the runner images](https://oreil.ly/1GyYt). The SBOM will have a filename of *sbom.<IMAGE-NAME>.json.zip* in the attachments of the release.

## Adding Additional Software on Runners

It is possible (and easy) to add additional packages on the GitHub-hosted runners. The basic process is just to create a job in the workflow that runs an appropriate package manager to install the tool you want, via a *`run`* invocation in a step. The step should just do what you would do if you were running the package management tool to install the package. For example, suppose you want to install a package on a Linux runner as part of your workflow. You could create a job like this:

```yml
jobs:
  update-env:
    runs-on: ubuntu-latest
    steps:
    - name: Install Package
      run: |
        sudo apt-get update
        sudo apt-get install <package-name>
```

You can use a similar process on a macOS runner using Brew as an example:

```yml
jobs:
  update-env:
    runs-on: macos-latest
    steps:
    - name: Install tree
      run: |
        brew update
        brew install --cask <package-name>
```

# Self-Hosted Runners

You have a choice of using the runners provided automatically by GitHub or hosting your own. Self-hosted runners are a useful option when you need more configurability and control over the environment(s) for executing your workflows. They provide a way to choose and customize the configuration, the system resources, and the available software that is used. Also, they provide a wider set of infrastructure options. They can be run on physical systems, on VMs, or in containers, either on-prem or in the cloud.

# Self-Hosted Runner Groups for an Organization

Self-hosted runners can be collected together into groups at the organization level. Organization admins can permit individual repositories access to a *runner group*, subject to managed policies.

Each organization has a single, default runner group. Runners are automatically assigned to this group at creation time. Additional groups can be created at the enterprise level.

For more information on runner groups, see [the enterprise doc](https://oreil.ly/u5QA-).

[Table 5-2](#advantages-of-self-ho) summarizes some of the key characteristics, advantages, and disadvantages of using self-hosted runners versus the ones provided by GitHub.

Table 5-2. Comparison of runner categories

| Category | GitHub-hosted | Self-hosted |
| --- | --- | --- |
| Provisioning/hosting | Managed by GitHub | Managed by you |
| Prereqs | Running GitHub actions runner application (handled by GitHub) | GitHub Actions self-hosted runner application deployed and configured |
| Platforms | Windows, Ubuntu Linux, MacOS | Any from the supported architectures and platforms that you choose |
| Configurability | Constrained mostly to predefined configurations | Highly configurable |
| Ownership | GitHub | As defined |
| Lifetime | Clean instance for the life of a job | As defined |
| Cost | Free minutes based on your GitHub plan with cost for overages | Free to use with actions, but owner is responsible for any other cost |
| Automatic updates | GitHub provides for the OS, installed packages, tools, and hosted runner application | GitHub provides only for self-hosted runner application |
| Implementation | Virtual | Virtual or physical |

For better correspondence with the way you use GitHub, runner systems can be mapped in (assigned) to different levels of GitHub repositories depending on the type of account you’re working with. The different levels and their mappings (as of the time of this writing) are shown in [Table 5-3](#github-account-scopes)

Table 5-3. GitHub account scopes

| Level | Scope of use |
| --- | --- |
| Repository | Intended to be used by a single repo |
| Organization | Intended for a GitHub organization (processing for multiple jobs across multiple repositories) |
| Enterprise | Assigned to multiple organizations for an enterprise account |

The mappings can be managed via going to the repository/organization/enterprise settings and adding the runner from there. But they can also be done via GitHub API calls. The API calls primarily focus on adding, deleting, or listing runners in the different levels of self-hosted runners in the previous table. See [the documentation](https://oreil.ly/OM_mm) for more information on the actual REST API calls.

## Requirements for Self-Hosted Runners

For a system to be used as a self-host runner for GitHub Actions, it must meet the following requirements:

* It must be based on a supported architecture and operating system. There is a more complete list [in the documentation](https://oreil.ly/_gqX6). However, this is basically modern versions of Linux, Windows, or macOS operating systems and x86-64 or ARM processor architectures.
* It must have the ability to run and install the self-hosted runner application from [*github.com/actions/runner*](https://github.com/actions/runner). The [*README.md*](https://oreil.ly/AgHi3) on the runner site contains links to releases and prerequisites.
* It must have the ability to communicate with GitHub Actions. At its core, this is the ability for the runner application to connect to GitHub hosts to download new versions of the runner and receive jobs that are targeted to the particular system. More details on the hosts at GitHub that may be commonly accessed can be found [in the documentation](https://oreil.ly/iyDQ5).
* It must have sufficient hardware resources (CPU, memory, storage, etc.) for the type of workflows you want to run.
* It must have appropriate software for the type of workflows and jobs you want to execute. (For workflows that use Docker container actions, a Linux machine with Docker installed is required.)
* It must have appropriate network access to needed or approved resources and endpoints.

## Limits for Self-Hosted Runners

There are limits imposed on Actions usage when you use self-hosted runners. As of the time of this writing, the [limits](https://oreil.ly/AAGm_) are those shown in [Table 5-4](#self-hosted-runner-li).

Table 5-4. Self-hosted runner limits

| Category | Limit | Action if limit is reached/exceeded |
| --- | --- | --- |
| Workflow run time | 35 days | Workflow canceled |
| Job queue time | 24 hours | Job terminated if not started |
| API requests | 1,000 per hour across all actions in a repository | Additional API calls will fail |
| Job matrix | 256 jobs per workflow run | Not allowed |
| Workflow run queue | 500 workflow runs per 10-second interval per repository | Workflow run terminated and fails to complete |
| Queuing by GitHub Actions | Within 30 minutes of being triggered | Workflow not processed (this would most likely only occur if GitHub Actions services are unavailable for an extended time) |

## Security Considerations for Using Self-Hosted Runners

You should not use self-hosted runners with public repositories. If someone forks your repo, they will also get a fork of the workflows and could do something potentially dangerous by initiating a pull request that would execute code in the forked workflow on your self-hosted runner.

This is especially dangerous if your system persists the environment between jobs. Remember that your self-hosted runner environment is only as secure as you make it. So, without proper safeguards, it could be affected by malicious code being executed, workflows reaching outside the system, or unapproved software or data being installed on the system.

Runners hosted via GitHub aren’t affected by this as they always create a clean, isolated environment that is then destroyed after the job is done.

You can manage network access to self-hosted runners through the same kinds of typical controls that you might use for other systems. For example, if you have a GitHub enterprise or organization account, you can have them go through [an *allowed list* for IP addresses](https://oreil.ly/e_v_0). And you can also use them with [a proxy server](https://oreil.ly/hRmdf).

# Step Permissions in Workflows

In some cases, you might encounter surprises as steps seem to have root access. Steps don’t run as root by default. But, as of the time of this writing, they do run as the same unprivileged user ID as the runner (agent) software. And that user does have password-less sudo to root as needed.

## Setting Up a Self-Hosted Runner

In this section, and the next, of the chapter, I’ll walk you through a simple example of setting up and using a self-hosted runner at the repository level.This will illustrate the simplest use case of using a local machine. But, you could also set up self-hosted runners in more complex on-prem environments or in cloud-based environments at the organization or enterprise level.

To begin, go to the repository’s *Settings* page from the top menu. Then, on the main Settings page, in the menu on the left, select *Actions* and then *Runners*, as shown in the left-side menu in [Figure 5-3](#runners-sub-menu).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0503.png)<br>
**Figure 5-3. Runners submenu**

You’ll see a large button in the upper right labeled New self-hosted runner. Clicking it will bring up a screen with options to choose for your new runner including the operating system and architecture. Based on your selections, you’ll get customized instructions on this screen to download the GitHub Actions Runner app, configure the system as a runner, and then use your self-hosted runner in your workflow for a job ([Figure 5-4](#adding-a-new-runner)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0504.png)<br>
**Figure 5-4. Adding a new runner**

From here, the process is straightforward—just follow the steps outlined in the *Download* and *Configure* sections to set up your runner machine. Clicking each step in that screen will also give you a copy icon (on the right) that you can click to copy and paste the command.

As part of the Configure section, there’s a shell script, `./config.sh`, that will start a local interactive configuration process on your machine. You can enter in custom values for each prompt or simply accept the defaults. Here is an example run:

```yml
developer@Bs-MacBook-Pro actions-runner %
./config.sh --url https://github.com/brentlaster/greetings-actions
--token ***********************

--------------------------------------------------------------------------------
|   ____ _ _   _   _       _          _        _   _                           |
|  / ___(_) |_| | | |_   _| |__      / \   ___| |_(_) ___  _ __  ___           |
| | |  _| | __| |_| | | | | '_ \    / _ \ / __| __| |/ _ \| '_ \/ __|          |
| | |_| | | |_|  _  | |_| | |_) |  / ___ \ (__| |_| | (_) | | | \__ \          |
|  \____|_|\__|_| |_|\__,_|_.__/  /_/   \_\___|\__|_|\___/|_| |_|___/          |
|                                                                              |
|                       Self-hosted runner registration                        |
|                                                                              |
--------------------------------------------------------------------------------

# Authentication

√ Connected to GitHub

# Runner Registration

Enter the name of the runner group to add this runner to:
[press Enter for Default]

Enter the name of runner: [press Enter for Bs-MacBook-Pro]

This runner will have the following labels:
'self-hosted', 'macOS', 'X64'
Enter any additional labels (ex. label-1,label-2):
[press Enter to skip]

√ Runner successfully added

√ Runner connection is good

# Runner settings

Enter name of work folder: [press Enter for _work]

√ Settings Saved.
```

After this, you execute the `./run.sh` script to have the runner start up and listen for jobs:

```yml
√ Connected to GitHub

Current runner version: '2.304.0'
2023-06-05 02:30:21Z: Listening for Jobs
```

Once you’ve executed this part of the process, your new runner should show up in the list of *Runners* on the *Settings* page ([Figure 5-5](#new-self-hosted-runne)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0505.png)<br>
**Figure 5-5. New self-hosted runner showing up in list**

## Using a Self-Hosted Runner

At this point, you’re ready to have jobs in your workflow use your new runner. You do this by specifying the `runs-on: self-hosted` clause. The following listing shows example code. Notice line 12 where the `runs-on` clause specifies `self-hosted`:

```yml
# Workflow to demo installing a package and executing on a self-hosted runner
name: file tree

on:
  workflow_dispatch:

jobs:
  file-tree:
    runs-on: self-hosted

    steps:
    - name: Install tree
      run: |
        brew update
        brew install tree

    - name: Execute tree
      run: time tree | tee filetreelist.txt
```

When we run this workflow through GitHub, the runner code on the local machine executes it. The following output is from a terminal on the runner machine:

```yml
 √ Connected to GitHub

Current runner version: '2.304.0'
2023-05-13 21:22:52Z: Listening for Jobs
2023-05-13 21:22:55Z: Running job: file-tree
2023-05-13 21:23:38Z: Job file-tree completed with result: Succeeded
```

Looking at the output for the single job in our workflow, it shows that the job was indeed run on the new runner, as shown in [Figure 5-6](#setup-job-run-on-self).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0506.png)<br>
**Figure 5-6. Set up job run on self-hosted runner**

## Using Labels with Self-Hosted Runners

Like the various *version-based* and *latest* labels that are configured by GitHub for GitHub-provided runners, a self-hosted runner automatically gets a set of labels when it is added to GitHub Actions. These labels include the following:

* *self-hosted*: default label applied to all self-hosted runners
* *linux, macOS,* or *windows*: applied based on OS
* *x64, ARM,* or *ARM64*: applied depending on architecture

If you want to apply a custom label to a self-hosted runner, you can pass it in when you run the initial config script—for example:

```yml
./config.sh --url <REPO_URL> --token <REG_TOKEN> --labels ssd,gpu
```

The script can also prompt you for additional labels when it is run. If you want to add a label later, you can go through the Settings > Actions > Runner menus for the organization or repository, then click the name of the runner, click the gear icon to edit, and add a new label there.

Note that when using labels in a workflow, they are cumulative. For example, the following declaration will run this job on a runner that has all three labels:

```yml
runs-on: [self-hosted, linux, ssd]
```

##### Runner Groups

Enterprise accounts or organizations using the Team plan can choose to add runners together into groups. Runner groups are used to collect sets of runners together with a security boundary around them. You can then choose which organizations or repositories are allowed to run jobs on a group. Organization administrators can also set access policies to control which repositories in an organization have access to a group.

Within a workflow, jobs can be identified to run on a particular group or on a particular group paired with labeled runners:

```yml
jobs:
  scans:
    runs-on:
      group: scan-runners
```

```yml
jobs:
  scans:
    runs-on:
      group: scan-runners
      labels: [self-hosted, linux, ssd]
```

You can find more about runner groups in the [documentation](https://oreil.ly/6wZUG).

## Troubleshooting Self-Hosted Runners

If GitHub and your self-hosted runner can’t communicate, the most obvious symptom will be that your job will not be scheduled and will appear to be stuck waiting on a runner.

Here’s example output for that case:

```yml
file-tree
Started 19181d 12h 14m 52s ago

Requested labels: self-hosted
Job defined at:
brentlaster/greetings-actions/.github/workflows/ostime.yml
@refs/heads/ostime
Waiting for a runner to pick up this job...
```

One of the first places to check to determine if there’s an issue is the Settings > Actions > Runner menu of the repository. If the runner in question shows up as Offline ([Figure 5-7](#self-hosted-runner-of)), there’s an issue keeping it from communicating with GitHub.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0507.png)<br>
**Figure 5-7. Self-hosted runner offline**

The problem could be as simple as the *run.sh* script no longer executing on the runner. If you need more info, the *run.sh* script includes a `--check` option that can be used to generate basic diagnostics. This option requires two key pieces of information:

* *URL:* the URL to the GitHub repository you’re working with
* *A personal access token (PAT):* a token generated via the Developer Settings that must have the *Workflow* scope (for more information, see [the GitHub tokens settings](https://oreil.ly/2EAMT))

Note that the PAT is different from the token that was used to configure the self-hosted runner.

Example output from running with the check option is shown next. This includes both a passing and failing check.

```yml
./run.sh --check --url https://github.com/brentlaster/greetings-actions --pat ghp_**

************************************************************************************
** Check:               Internet Connection
** Description:         Check if the Actions runner has internet access.
************************************************************************************
**                                                                                **
**                                    P A S S                                     **
**                                                                                **
************************************************************************************
** Log: /Users/developer/actions-runner/_diag/InternetCheck_20220709-184812-utc.log
************************************************************************************

************************************************************************************
** Check:               GitHub Actions Connection
** Description: Check if the Actions runner has access to the GitHub Actions service
************************************************************************************
**                                                                                **
**                                    F A I L                                     **
**                                                                                **
************************************************************************************
** Log: /Users/developer/actions-runner/_diag/ActionsCheck_20220709-184812-utc.log
** Help Doc: https://github.com/actions/runner/blob/main/docs/checks/actions.md
************************************************************************************
```

Notice that there are printed links to logs where you can get more information. For the case that failed, there is also a reference for a help doc.

Digging into the log reveals more information about what detailed checks were run and which ones succeeded/failed for that group:

```sh
cat /Users/developer/actions-runner/_diag/ActionsCheck_20220709-184812-utc.log
...
...
2022-07-09T18:48:12.8336080Z ***************************************************
2022-07-09T18:48:12.8336090Z ****                                           ****
2022-07-09T18:48:12.8336090Z **** Try ping pipelines.actions.
githubusercontent.com
2022-07-09T18:48:12.8336100Z ****                                           ****
2022-07-09T18:48:12.8336100Z ***************************************************
2022-07-09T18:48:17.8521990Z 
Ping pipelines.actions.githubusercontent.com (0.0.0.0) failed with 
'TimedOut'
```

## Removing a Self-Hosted Runner

Depending on your needs and your access, there are different ways to remove a self-hosted runner.

If you only need to temporarily stop jobs from being assigned to one of your runners, you can simply stop the *run* application or shut down the system. In this case, you will see the machine still being assigned in the Runners list but in an *Offline* state (see [Figure 5-7](#self-hosted-runner-of)). It will stay in this state until the runner app is restarted again via the *run* application. If the system doesn’t get connected to GitHub Actions for more than 30 days, it will automatically be removed.

# Automatic Failure

If you have jobs trying to execute and the runner machine is not available, eventually your job will fail ([Figure 5-8](#job-failed-after-runn)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0508.png)<br>
**Figure 5-8. Job failed after runner unavailable for one day**

The process for removing a self-hosted runner varies slightly depending on whether you are removing it from a single repository, an organization, or an enterprise. But the basic steps are to go to the *Settings* page, select *Actions,* and then select *Runners*. Then click the name of the runner you want to remove. You’ll see a screen like the one in [Figure 5-9](#option-to-remove-self).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0509.png)<br>
**Figure 5-9. Option to remove self-hosted runner**

From this screen, you click the Remove button. You may be prompted for your password, and then you’ll see a list of instructions for removing the runner. The process will vary depending on whether or not you have access to the system. If you do have access, follow the instructions for the removal process. These instructions will have a URL and a temporary token you can use. This process will remove the configuration data from the machine and will also remove the runner from GitHub. The following is an example output from running the remove command:

```sh
developer@Bs-MacBook-Pro actions-runner % ./config.sh remove
 --token AARNGCCLH3PFFSFSHVFOCCLCZHTDO

# Runner removal

√ Runner removed successfully
√ Removed .credentials
√ Removed .runner
```

If you don’t have access to the machine, you can still make GitHub remove it from the list of the registered runners by clicking Force remove this runner (see [Figure 5-10](#remove-runner-options)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0510.png)<br>
**Figure 5-10. Remove runner options**

After this process, you should see that the runner is no longer listed.

Finally, in this chapter, I’ll cover a couple of advanced self-hosted runner topics: autoscaling and just-in-time runners.

# Autoscaling Self-Hosted Runners

There a couple of ways you can autoscale your self-hosted runners.

If you have access to a [Kubernetes cluster](https://kubernetes.io), you can set up self-hosted runner orchestration and scaling there via the *Actions Runner Controller* (ARC). ARC works as a [Kubernetes operator](https://oreil.ly/tdVeG) to create *scale sets.* Scale sets are a group of homogeneous runners that are controlled by the ARC and can have jobs assigned to them from GitHub Actions. The scale sets can automatically scale self-hosted runners based on the number of workflows running in a repository, an organization, or an enterprise. The ARC can be installed via the Kubernetes orchestration and packaging tool [Helm](https://helm.sh). For more information on using the ARC for autoscaling, see the [quickstart documentation](https://oreil.ly/wc852).

Alternatively, on Amazon Web Services, there’s a [Terraform web module](https://oreil.ly/7tw9f) for scalable runners on that platform. However, GitHub is officially recommending the Kubernetes approach for users who want to do autoscaling.

# Just-in-Time Runners

Autoscaling is only recommended if you are *not* using persistent runners. *Persistent* means runners that stay around across runs of multiple jobs. Persistence is the default behavior for self-hosted runners.

To make self-hosted runners not persistent, you can simply supply the `--ephemeral` flag at the time you configure them. When you create a runner as *ephemeral*, GitHub only assigns one job to a runner. This means your runners act more like GitHub-hosted runners, providing a clean environment for each job. Here’s an example of configuring a runner as ephemeral:

```sh
./config.sh --url https://github.com/brentlaster/greetings-actions
--token *********************** --ephemeral
```

You can also create ephemeral, just-in-time (JIT) runners by using the REST API to [create the configuration for a JIT runner](https://oreil.ly/01mXq). After you have the config file from the REST API call, you can pass it on to the runner at startup:

```sh
./run.sh --jitconfig ${encoded_jit_config}
```

These self-hosted runners will execute only one job before they are automatically removed.

# Conclusion

Runners provide the required infrastructure to execute your workflows and thus GitHub Actions. Runners can be automatically provided by GitHub through their hosting, or you can download the Runner app and use your own systems as runners. There are advantages and disadvantages to each, including key factors such as cost, maintenance, control, configurability, and simplicity. Hosted runners are available for Ubuntu Linux, Windows, and macOS for preselected operating system versions and on standardized virtual systems. GitHub periodically updates and maintains these standardized environments.

Your workflows can choose a particular runner through the `runs-on` clause for each job in your workflow. You can also utilize standard OS commands (like calls to *apt* or *brew*) to install additional software on the systems if needed.

Self-hosted runners can be made ephemeral to only execute one job. This is desirable for autoscaling solutions such as the ARC.

The next chapter will introduce the *building blocks* section of the book to help you understand how to build out your workflows and related pieces.

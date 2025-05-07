# Chapter 6. Managing Your Workflow Environments

Beyond the basic structure and components covered in [Part I](part01.html#foundations) of this book, GitHub Actions offers a rich set of functionality to build out and support your automation. In this section of the book, I’ll cover some key areas that you’ll want to understand and manage to get the most functionality out of the workflows you create.

In this chapter, I’ll focus on the items that you can manage and leverage to define the environment that your workflow uses. The topics covered here include the following:

* Naming your workflow and workflow runs
* Contexts
* Environment variables
* Secrets and configuration variables
* Managing permissions for your workflows
* Deployment environments

I’m going to start with the most straightforward—naming your workflow and workflow runs.

# Naming Your Workflow and Workflow Runs

[Chapter 4](ch04.html#ch04) referenced the name of a workflow as part of the coding examples. In the workflow syntax, GitHub Actions provides keywords that allow you to name both your workflow and the runs of the workflow. You can use the `name` keyword to set the displayed name of your workflow on the *Actions* tab, as in:

```yml
name: Pipeline
```

This is surfaced in the Actions tab, as shown in [Figure 6-1](#workflow-with-name-se).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0601.png)<br>
**Figure 6-1. Workflow with name set**

If you don’t set this value, GitHub Actions will set it to be the name of the workflow file, relative to the root of the repository.

You can also provide a naming pattern for the runs of your workflow based off of data provided by GitHub:

```yml
run-name: Pipeline run by @${{ github.actor }}
```

[Figure 6-2](#workflow-run-with-a-c) shows what this string looks like for a change made by the *gwstudent2* ID.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0602.png)<br>
**Figure 6-2. Workflow run with a customized name**

In the preceding example, I’m setting the `run-name` value to include the *actor* property from the *github* *context*. Contexts are very useful data values available to leverage in your workflows. And they are the subject of the next section.

# Contexts

In GitHub Actions, contexts are collections of properties related to a specific category, such as the runner, GitHub data, jobs, secrets, etc. They are provided via Actions and generally available for you to use in your workflows. The `github.actor` reference in the previous section is one example. In this case, the context is `github`, and the specific property is `actor`—the username of the user that triggered the initial workflow run. The properties are usually strings but may be other objects.

The availability of a context may vary depending on what is occurring in the workflow. For example, there is a *secrets* context (which allows your workflow to access the values of secrets) that is only available at certain points during job execution. Likewise, there is a *matrix* context which is only available when you are using the matrix job strategy (discussed in Chapters [8](ch08.html#ch08) and [12](ch12.html#ch12)). More detailed info on when certain contexts and their functions are available can be found [in the documentation](https://oreil.ly/MZwU7).

A high-level listing of the different contexts and their purposes is shown in [Table 6-1](#overview-of-contextsc).

Table 6-1. Overview of contexts

| Context | Purpose | Example properties |
| --- | --- | --- |
| [`github`](https://oreil.ly/DDqr8) | Data attributes about the workflow run and the event that triggered the run. | `github.ref`  `github.event_name`  `github.repository` |
| [`env`](https://oreil.ly/1ySBL) | Variables that have been set in a workflow, job, or step. | `env.<env_name>` (to retrieve value) |
| [`vars`](https://oreil.ly/Sfd3j) | Configuration variables set for a repository, environment, or organization (see also [Chapter 12](ch12.html#ch12)). | `vars.<var_name>` (to retrieve value) |
| [`job`](https://oreil.ly/LaeZE) | Information about the currently running job. | `job.container`  `job.services`  `job.status` |
| [`jobs`](https://oreil.ly/NOqqE) | Only available for reusable workflows. Used to set outputs from reusable workflows. | `jobs.<job_id>.results`  `jobs.<job_id>.outputs` |
| [`steps`](https://oreil.ly/6Odjx) | If a step has an id property associated with it and has already run, this contains information from the run. | `steps.<step_id>.outcome`  `steps.<step_id>.outputs` |
| [`runner`](https://oreil.ly/6mR2y) | Information about the runner executing the current job. | `runner.name`  `runner.os`  `runner.arch` |
| [`secrets`](https://oreil.ly/wB6rW) | Contains names and values associated with a secret; not available in composite workflows but can be passed in. | `secrets.GITHUB_TOKEN`  `secrets.<secret_name>` (to retrieve value) |
| [`strategy`](https://oreil.ly/lqIBd) | If a matrix is used to define a set of items to execute across, this context contains information about the matrix for the current job. | `strategy.job-index`  `strategy.max-parallel` |
| [`matrix`](https://oreil.ly/idKO8) | For workflows that use a matrix, contains the matrix properties that apply to the current job. | `matrix.<property_name>` |
| [`needs`](https://oreil.ly/TqEZr) | Used to collect output from other jobs; contains output from all jobs that are defined as a direct dependent of the current job. | `needs.<job_id>`  `needs.<job_id>.outputs`  `needs.<job_id>.outputs.<output name>` |
| [`inputs`](https://oreil.ly/_W0x2) | Contains input properties that are passed in to an action, a reusable workflow, or a manually triggered workflow. | `inputs.<name>` (to retrieve value) |

You can reference any context property through standard GitHub Actions expression syntax such as `${{ context.property }}`. The contexts can also be leveraged as part of conditional expressions such as `if: ${{ github.ref == 'ref/heads/main' }}`. In this example, the code checks if the current branch is *main*.

# Untrusted Input in Contexts

Be aware that certain context properties may be subject to being modified from their original value. An example would be input parameters that have had code injected into them. Therefore, some context properties should be treated as untrusted input and a potential security risk. [Chapter 9](ch09.html#ch09) on security describes the applicable situations in more detail and how to prevent being affected by these cases.

Most contexts allow you to get predefined data from key categories that are made available to you automatically. But the *env* context allows you to easily specify your own data to use in your workflow through defining custom environment variables.

# Environment Variables

Within a workflow, you can define environment variables to be used at the level of a workflow, an individual job, or even an individual step. To set these up, use an *env* section. The `env` section is a mapping of variables to values, stored in the *env* context. Here is an example:

```yml
# workflow level
env:
  PIPE: cicd

# job level
jobs:
  build:
    env:
      STAGE: dev

# step level
  steps:
    - name: create item with token
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Per the last line, note that you can use context values as the values for the variable.

As shown in the listing, you can have multiple levels of variables—at the workflow, job, and step levels. If the same variable exists at multiple levels, variables defined at the step level override variables defined at a job or workflow level. And variables defined at the job level override variables defined at the workflow level.

Technically, environment variables you define within a workflow are called *custom environment variables.* This is to distinguish them from a set of provided *default environment variables* that GitHub Actions also makes available.

## Default Environment Variables

GitHub provides a default set of environment variables available for workflows to use. These will be named starting with *GITHUB_* or *RUNNER_*. Examples include `GITHUB_WORKFLOW`, which is set to the name of the currently running workflow, and `RUNNER_OS`, which is set to the type of OS executing a job. The complete set of default environment variables can be found [in the documentation](https://oreil.ly/9imlG).

# Default Variable Lifetime

It’s important to note that the default environment variables only exist on the runner system, whereas contexts are available even before the job gets to a runner.

These variables can be used together in workflows to get information at runtime. For example, here’s a simple job to report the URL of a workflow run:

```yml
jobs:

  report-url:
    runs-on: ubuntu-latest
    steps:
      - run: echo $GITHUB_SERVER_URL/$GITHUB_REPOSITORY/actions/runs/$GITHUB_RUN_ID
```

Running this code produces output like the following that includes the URL that takes you back to that run:

```yml
echo $GITHUB_SERVER_URL/$GITHUB_REPOSITORY/actions/runs/$GITHUB_RUN_ID
https://github.com/gwstudent2/greetings-ci/actions/runs/4744932978
```

Most of the default environment variables have corresponding properties that can be used from the *github* or *runner* contexts. For example, the preceding code that used the environment variables could also be written using context properties as follows:

```yml
jobs:

  report-url:
    runs-on: ubuntu-latest
    steps:
      - run: echo ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

As one more example, you can use the default environment variable *RUNNER_OS* or the context property *runner.os* to get/display the OS that the workflow is running on. Given this code:

```yml
  report-os:
    runs-on: ubuntu-latest
    steps:
      - name: check-os
        if: runner.os != 'Windows'
        run: echo "The runner's operating system is $RUNNER_OS."
```

the following output would be produced:

```yml
The runner's operating system is Linux.
```

Since I’ve mentioned default environment variables here, it’s worth noting that you can set default values at the workflow or job levels for two *system* environment settings—the shell and working directory. An example is shown here:

```yml
on:
  push:

defaults:
  run:
    shell: bash
    working-directory: workdir

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: sh
        working-directory: test
    steps:
      - uses: actions/checkout@v4
      - run: echo "in test"
```

When defaults are specified at both levels, the job settings will override the workflow settings.

The environment variables shown so far have been used in the context of single workflows. But it is also possible to define values for a repository, an organization, or an environment that can be used and accessible for multiple workflows. Those values fall into two very similar use cases—*secrets* for data that should be hidden/encrypted and *configuration variables* for nonsensitive data.

# Secrets and Configuration Variables

Earlier I noted the different types of contexts that are available for you to use in your workflows. Among these was the *secrets* context for referencing secure data values stored in GitHub. Storing data that should not be exposed (such as an access token) in a secret is a best practice so you are not exposing that data in your workflow. Actions also has special handling built in for working with secrets, such as masking them in log output so they are not printed.

*Configuration variables* (aka *repository variables*) are similar to secrets except they’re not intended to be used for secure data. They can be used to hold any kind of setting/value that is OK to expose and needs to be set at the repository or organization level.

Regardless of whether you need to use a secret or a configuration variable, the process to set them up is nearly the same. Of course first, you must have access to do this at the repository, organization, or environment level. Then, to create a secret or variable to be available to workflows, you can follow these steps:

1. Go to the *Settings* for your repository.
2. On the lefthand menu, in the *Security* section, click *Secrets and variables*.
3. Click *Actions*.
4. Click the appropriate tab for *Secrets/Variables*.
5. Click the New secret/New variable button.
6. Fill in the *Name* and *Secret/Value* fields with the appropriate data.
7. Click the Add secret/Add variable button to save your item.

Here’s a simple example to show you how a configuration variable can be defined and accessed.

First, go to the *Settings* tab for the organization, select the *Security* section, then *Secrets and variables*, and then the *Actions* option ([Figure 6-3](#getting-to-the-option1)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0603.png)<br>
**Figure 6-3. Getting to the option to set configuration variables**

After that, select the *Variables* tab and click the New organization variable button ([Figure 6-4](#actions-variables-co1)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0604.png)<br>
**Figure 6-4. Actions variables (configuration variables) tab**

From here, define the variable. In this case, I’m calling it *FILE_TO_CHECK* and giving it an initial value of *CONTRIBUTING.md*. [Figure 6-5](#adding-a-new-configur1) shows the add screen.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0605.png)<br>
**Figure 6-5. Adding a new configuration variable**

At the bottom of the screen is a *Repository access ** option. This option allows you to select the scope of repositories that the variable will be in effect for ([Figure 6-6](#configuration-variabl1)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0606.png)<br>
**Figure 6-6. Configuration variable scope**

Choosing the Selected repositories option results in an option to select individual repositories ([Figure 6-7](#icon-to-select-indivi)) via the gear icon.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0607.png)<br>
**Figure 6-7. Icon to select individual repositories**

This brings up a dialog listing repositories that are available to have the variable apply to ([Figure 6-8](#repositories-to-selec1)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0608.png)<br>
**Figure 6-8. Repositories to select from for configuration variable to work with**

You can also set a configuration variable for a specific repository. The process works the same but will not, of course, have the option to have the variable apply to other repositories.

Now suppose you want to define a simple workflow to verify a file. [Figure 6-9](#set-of-defined-variab1) shows a list of variables you might define for that. EXEC_WF is a *switch* to say whether or not to execute the workflow. This provides a way to temporarily turn off the required workflow if you want. FILE_TO_CHECK defines the name of the file to verify exists. And JOB_NAME simply illustrates other places and formats that can be used for variables.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0609.png)<br>
**Figure 6-9. Set of defined variables**

A variable created at this level can be referenced via the vars context (`${{ vars.​VARI⁠ABLE_​NAME }}`). The secrets created in the repository can also be referenced in the workflow via the *secrets* context (`${{ secrets.SECRET_NAME }}`). [Chapter 9](ch09.html#ch09) goes into more details on using secrets in your workflows as part of a broader security strategy.

Here’s a workflow that uses the variables I’ve defined:

```yml
 1 name: Verify file
 2
 3 on:
 4   push:
 5   pull_request:
 6
 7 jobs:
 8   verify:
 9     name: ${{ vars.JOB_NAME }}
10     if: ${{ vars.EXEC_WF == 'true' }}
11     runs-on: ubuntu-latest
12
13     steps:
14       - uses: actions/checkout@v4
15       - run: |
16           [[ -f ${{ vars.FILE_TO_CHECK }} ]] ||
( echo "${{ vars.FILE_TO_CHECK }} file needs to be added to
${{ github.repository }} !" && exit 1 )
```

In the listing, lines 9, 10, and 16 show the use of the configuration variables. When this workflow is run, the values are substituted appropriately, as shown in [Figure 6-10](#workflow-run-with-var1).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0610.png)<br>
**Figure 6-10. Workflow run with variables substituted**

As discussed, configuration variables are different from the environment variables explained in the preceding section. Environment variables are intended for use only in the scope of a workflow. However, both types can be used within your workflow. Here’s an example of combining both types of variables:

```yml
env:
  INFO_LEVEL: ${{ vars.INFO_LEVEL }}
```

Secrets and the different types of variables allow you to define values you want your workflow to use. But to allow your workflow to work with other types of resources, you may also need to adjust the permissions available to the workflow.

# Managing Permissions for Your Workflow

If your workflows need to perform operations that produce and/or change content in your repository, they will need to have the appropriate permissions. The way that GitHub Actions allows your workflows to do this is by installing a special app in a repository if Actions is enabled. This app brings along with it an installation access token referred to as the *GITHUB_TOKEN*. This token is stored as a secret and is used to authenticate on behalf of the GitHub App installed in your repository. Its access is limited to the single repository with your workflows.

By default, the GITHUB_TOKEN has a core set of permissions allowing it to perform functions in the repository. Administrators for a repository, organization, or enterprise can set the overall default access to be either permissive or restrictive. [Table 6-2](#default-github_token) (excerpted from the GitHub Actions documentation) shows the default access for each category and access type.

Table 6-2. Default GITHUB_TOKEN permissions

| Scope | Default access  (permissive) | Default access  (restricted) |
| --- | --- | --- |
| actions | read/write | none |
| checks | read/write | none |
| contents | read/write | read |
| deployments | read/write | none |
| id-token | none | none |
| issues | read/write | none |
| metadata | read | read |
| packages | read/write | read |
| pages | read/write | none |
| pull-requests | read/write | none |
| repository-projects | read/write | none |
| security-events | read/write | none |
| statuses | read/write | none |

In basic workflows, the default permissions may be sufficient to accomplish what’s needed. However, sometimes you may need to modify the permissions to allow or deny additional access. To do that, you use the *permissions* keyword and add the *scope: permissions* format. For example, if you were running under the overall restricted access model and wanted to add permissions for the workflow to create issues, you could add the following code in your workflow:

```yml
permissions:
  issues: write
```

This code could be added either in the main body of the workflow to provide all jobs with access or within individual jobs to provide only that job access. To maintain the best security, only augment the permissions where absolutely required. [Chapter 9](ch09.html#ch09) on security goes into more detail on this.

Aside from augmenting the permissions, there are use cases where you may need to pass the token for other functions. Generally, these fall into one of two categories:

* Passing the token as input to an action that requires it:

  ```yml
      steps:
        - uses: actions/labeler@v5
          with:
            repo-token: ${{ secrets.GITHUB_TOKEN }}
  ```

* Using the token to invoke other functionality via REST API calls:

  ```yml
     --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}'
  ```

In both examples, the token is referenced through the context *secrets*. If you need more/different access than the GITHUB_TOKEN can provide, you can also [create a personal access token](https://oreil.ly/A0c8S) and store it as a secret in the repository to use in the same way in your workflow.

In the final section of this chapter, I’ll look at a way not only to manage the data values and permissions of your workflow but to define distinct environments for different types of deployment activities. That section will also show you the ways to control when and how jobs referencing those environments execute.

# Deployment Environments

*Environments* in GitHub Actions are objects used to identify a general target for deployment. An example might be a *level* like *dev*, *test*, or *production*. A job in a workflow can reference one (and only one) such environment. This reference then identifies a target for any deployment steps in the job.

These environments can be configured with their own data values (secrets and variables) restricted to that environment. Those values are different from repository or organization secrets and variables. Environments can also be configured with restrictions on what has to occur before jobs referencing them are allowed to run. These are referred to as *deployment protection rules*. Generally, environments can only be configured for public repositories. But users of GitHub Pro and organizations using GitHub Team can configure environments for private repositories.

Deployment protection rules act like gates that must be passed before jobs referencing them can proceed. If one of your workflow jobs references an environment, the job won’t start until all of the environment’s protection rules have passed. Example use cases for this could be to restrict an environment to certain branches, delay jobs, or require manual approval. The deployment protection rules available to you *out of the box* include the following:

> *Required reviewers*
>> Allows you to require up to six people or teams as reviewers that must approve workflow jobs
>
> *Wait timer*
>> Allows you to specify a number of minutes (0–43,200) to delay a job after it is initially triggered (43,200 minutes = 30 days)
>
> *Deployment branches*
>> Allows you to restrict which branches can deploy to the environment from these choices:
>>
>> * All branches
>> * Protected branches—only branches with branch protection rules
>> * Selected branches—must match name patterns you specify

Environments are defined in the Settings for a repository. Each environment, along with any secrets, variables, and/or protection rules, is defined separately.

It is also possible to create custom deployment protection rules with third-party services. These could be useful to approve/reject deployments based on data such as security scanning results from a third party, whether a ticket is approved, etc. To create such rules, you need to be familiar with GitHub Apps, webhooks, and callbacks. More information can be found in [the documentation](https://oreil.ly/_MBf0).

# Custom Deployment Protection Rules Status

As of the time of this writing, custom deployment protection rules are still in *beta* and subject to change.

To provide a better understanding of how environments work with GitHub Actions, next up is some example code and screenshots showing how to configure an environment.

Starting with the main code body, shown in the following listing, you have the basic trigger for a push on either the branch *main* or the branch *dev.* Then there’s a job to check out code from the repository and build and test it using Gradle. After that work, the output of the build is uploaded as an artifact to persist it for the other jobs in the workflow. If you’ve followed along with the earlier chapters of the book, this should be pretty straightforward to understand:

```yml
name: Deployments example
on:
  push:
    branches: [ "main", "dev" ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'

    - uses: gradle/gradle-build-action@v2
      with:
        arguments: build
    - uses: gradle/gradle-build-action@v2
      with:
        arguments: test

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: archive.zip
        path: build/libs
```

The next listing contains a continuation of the code in the workflow. There’s a job for deploying code to a dev environment. Notice the `if` clause near the top that only allows this job to run if the branch being pushed to is *dev*. After that is the association to the dev environment. A `url` to deploy assets to the dev environment is also supplied. That’s followed by a download of the persisted artifact.

Notice that there are several references to a variable named `DEV_VERSION` as well as a `DEV_TOKEN` secret. Both of these values are set in the environment configuration and accessible only to jobs that reference the dev environment.

Finally, in the job, there’s a call to an action named [softprops/action-gh-release](https://oreil.ly/juW-o), which helps you create GitHub releases:

```yml
  deploy-dev:

    needs: [build-and-test]
    if: github.ref == 'refs/heads/dev'

    runs-on: ubuntu-latest
    environment:
      name: dev
      url: https://github.com/${{ github.repository }}/releases/tag/v${{ vars.DEV_VERSION }}

    steps:
      - name: Download candidate artifacts
        uses: actions/download-artifact@v4
        with:
          name: archive.zip

      - name: release to dev
        uses: softprops/action-gh-release@v0.1.15
        with:
          tag_name: v${{ vars.DEV_VERSION }}
          token: ${{ secrets.DEV_TOKEN }}
          prerelease: true
          draft: true
          name: dev
          files: greetings-deploy.jar
```

The last job (next listing) of the workflow is very similar to the *deploy-dev* job, except it is for deploying to the production environment instead. You can see the same type of `environment` definition.

In this job, there is also a variable (*PROD_VERSION*) and a secret (*PROD_TOKEN*) particular to this production environment. Having different versions for the jobs allows for a dev version of the artifact and also a production version with different version numbers configured for the two different environments. The same applies for the tokens. While you might think that a token would typically be the same, consider that you might want some differences in key areas. For example, you may want the dev token to have a broader set of scopes or the production token to have a shorter expiration time frame:

```yml
  deploy-prod:

    needs: [build-and-test]
    if: github.ref == 'refs/heads/main'

    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://github.com/${{ github.repository }}/releases/tag/v${{ vars.PROD_VERSION }}

    steps:
      - name: Download candidate artifacts
        uses: actions/download-artifact@v4
        with:
          name: archive.zip

      - name: GH Release
        uses: softprops/action-gh-release@v0.1.15
        with:
          tag_name: v${{ vars.PROD_VERSION }}
          token: ${{ secrets.PROD_TOKEN }}
          generate_release_notes: true
          name: Production
          files: greetings-deploy.jar
```

Environments can be created/edited through the Settings/Actions menu in a repository. [Figure 6-11](#creatingediting-an-e) shows the top part of the creation/editing screen for an environment. In this case, for the production environment, I’ve added one required reviewer.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0611.png)<br>
**Figure 6-11. Creating/editing an environment, part 1**

[Figure 6-12](#creatingediting-an-e2) shows the second half of the configuration screen for the production environment. Notice that in this section, I’ve added the *PROD_TOKEN* secret and the *PROD_VERSION* variable that are exclusive to this environment.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0612.png)<br>
**Figure 6-12. Creating/editing an environment, part 2**

When I make a change and push it out on the *main* branch, the deployment protection rules are activated. On the summary page for the job, since I set one of the rules up to require a review by another user, I’ll see a message to that effect ([Figure 6-13](#production-environmen1)). The job is blocked from starting until the review is completed.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0613.png)<br>
**Figure 6-13. Production environment requires review**

The designated reviewer will get an email ([Figure 6-14](#email-request-to-revi)) informing them that there’s an environment waiting for their review.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0614.png)<br>
**Figure 6-14. Email request to review**

The reviewer can go to the review, leave a comment, and then choose to reject or approve the deployment, as shown in [Figure 6-15](#approving-a-pending-d).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0615.png)<br>
**Figure 6-15. Approving a pending deployment**

If the deployment is approved by the reviewer, the job associated with the production environment will be unblocked, and the deployment activities will continue. See the job graph in [Figure 6-16](#deployment-approved) for an example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0616.png)<br>
**Figure 6-16. Deployment approved**

Back on the Actions summary page, there will be a new Deployments item on the left ([Figure 6-17](#deployment-shortcut-i)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0617.png)<br>
**Figure 6-17. Deployment shortcut in workflow runs**

Clicking this will take you to the history of deployments from the workflow, as shown in [Figure 6-18](#deployment-history).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0618.png)<br>
**Figure 6-18. Deployment history**

And, clicking the View deployment button takes you to the actual release with the corresponding assets ([Figure 6-19](#production-assets-ava)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0619.png)<br>
**Figure 6-19. Production assets available**

Assuming I have configured the dev environment appropriately, a similar flow would happen for a push to the dev branch. For the dev environment, I could choose to configure it with different protection rules (such as no reviewers since it’s dev), a different version number scheme, and/or a different token if I wanted.

If I didn’t specifically configure the dev environment in advance, GitHub would create the environment for the run. But in this case the run would fail because a token hadn’t been explicitly added to the environment.

# Conclusion

GitHub Actions provides several ways to get and set properties of the environment for use in your workflows. Contexts provide data properties associated with key categories such as GitHub, runners, secrets, and more.

Environment variables can be used to set values for use in a single workflow and referenced via the env context. Configuration variables can be set at the repository or organizational level and provide mappings for use across workflows. Secrets serve a similar purpose but encapsulate data that needs to be handled securely.

When your workflow needs to access key information or interact more directly with the repository, you may need to adjust the permissions it has. This can be done by assigning more permissions to the built-in GITHUB_TOKEN. Permissions should be the minimum needed to allow for the workflow or job to accomplish what is needed.

Deployment environments give you a way to provide destinations to deploy items from your workflow into separate areas. You can configure protection rules, tokens, and variables that are unique to the environment and associate an environment to a job. This setup allows you to exercise more control over what conditions can be used to run your jobs, provide exclusive data values, and provide a way to differentiate your workflow runs for different levels, such as *dev, test, prod*.

In the next chapter, we’ll look at how to manage data that is created and accessed during the workflow runs.

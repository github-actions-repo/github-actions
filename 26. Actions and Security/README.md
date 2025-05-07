# Chapter 9. Actions and Security

As seen throughout the preceding chapters, actions provide impressive levels of automation. They also provide ways to accomplish tasks in GitHub directly that would not be possible otherwise. However, these same capabilities can also imply security risks that must be considered and planned for in advance. Otherwise, you are opening your repositories up to multiple attack surfaces and vulnerabilities. This can be either through someone taking deliberate advantage of security holes or through accidental misuse. And you are opening up the repository of anyone who forks yours to the same kinds of exposures.

Keep in mind that you are using a framework wholly designed for collaboration. While GitHub provides world-class [security](https://oreil.ly/Hi4sb) for its platform and data, it is still up to the individual repository owners to take the appropriate precautions and measures to secure their repositories. This includes managing *who* and *what* is allowed to operate within them. This is especially important with workflows and actions in the mix since the specific purpose of them is to execute code.

In this chapter, I’ll look at the security implications of working with workflows and actions in the context of your repositories. And I’ll review the mechanisms that GitHub provides to allow you to set appropriate bounds on what your actions can do and when they can be executed. Throughout, I’ll also highlight some best practices from GitHub around security with workflows and actions.

Securing the use of actions requires a multilayered approach. There are many ways to look at those layers, but the simplest approach is the following:

> *Security by configuration*
>> Implementing appropriate controls and settings to govern what can run and when
>
> *Security by design*
>> Leveraging tokens and secrets to secure data; guarding against common threats such as untrusted input; securing dependencies
>
> *Security by monitoring*
>> Reviewing changes especially when coming through pull requests; scanning; monitoring execution

A good place to start is taking advantage of GitHub’s security options for your repositories.

# Security by Configuration

As it pertains to the actions framework, security by configuration is about these conditions:

* Whether or not actions and workflows are allowed to run at all
* If allowed, what the criteria are for which they can be run

Configuration for actions and workflow options is done by going to the *Settings* tab for the repository, then to the *Actions* menu on the left side, and selecting the *General* option ([Figure 9-1](#the-main-actions-perm)). (This assumes you have permissions to modify the Settings for the repository.)

# Actions Runners Menu Item

Note that the other option under the Actions menu is the *Runners* option. This is where you can configure your own self-hosted runners. Runners are covered in [Chapter 5](ch05.html#ch05).

Once you’ve selected the *General* option, at the top of the page is a set of options to specify which actions and workflows you will allow for this repository.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0901.png)<br>
**Figure 9-1. The main actions permissions screen**

Currently the first item in this list is the *Actions permissions* section. This is where you can allow all actions, disable them completely, limit actions and workflows to only those in repositories owned by the current user, or allow those owned by the current user *and* that match specific criteria. (A common element of these is reusable workflows. Reusable workflows are discussed more in [Chapter 12](ch12.html#ch12).)

The first three options can be understood by reading the associated text. The last option deserves some additional explanation. When you select that option ([Figure 9-2](#allowing-actions-and22)), you’re given a way to specify the different criteria.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0902.png)<br>
**Figure 9-2. Allowing actions and reusable workflows**

The first checkbox on this screen, for *actions created by GitHub*, refers to ones you’d find under [*https://github.com/github/actions*](https://github.com/github/actions). These are actions that have been provided by GitHub itself ([Figure 9-3](#actions-provided-by-g1)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0903.png)<br>
**Figure 9-3. Actions provided by GitHub**

The second checkbox is to allow actions by Marketplace-*verified creators*. As of the time of this writing, the term *verified creator* here means that the creator of the action has been verified by GitHub’s business development team.

Clicking the *verified creators* link in that option will take you to the [Actions Marketplace page](https://oreil.ly/KXe9o), with the listing filtered by actions that have verified creators. Those actions will have the small icon that looks like a gear with a checkmark in it next to the author (see [Figure 9-4](#verified-creators-ide)). All actions created by GitHub itself are verified.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0904.png)<br>
**Figure 9-4. Verified creators identifier**

Below that is a free-form text entry box where you can enter a comma-separated list of actions and workflows that you want to allow. Examples for different types of entries are shown below the box, but for actions, it follows the standard syntax you would have in the `uses` statement in a workflow. For workflows themselves, you can specify the full paths that include the `.github/workflows` directory ([Figure 9-5](#specifying-allowed-ac)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0905.png)<br>
**Figure 9-5. Specifying allowed actions**

After this section, there is an area to change the artifact and log retention period from the default of 90 days. (Artifacts are discussed in [Chapter 7](ch07.html#ch07).)

## Managing Execution of Workflows from Pull Requests

The next section on the page allows you to manage which outside collaborators can run workflows on pull requests for your repository. The idea here is that you don’t want to necessarily allow everyone who forks your repository to execute the workflows you’ve defined in it ([Figure 9-6](#managing-which-collab)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0906.png)<br>
**Figure 9-6. Managing which collaborators can run workflows on PRs for your repo**

# Who Is an Outside Collaborator?

If you have an organization on GitHub, an outside collaborator is anyone who is not a member of your organization but has been given access to a repository in your organization by an admin for the repository.

Think of it this way: your workflow(s) were designed for your repository. They may rely on specific content or inputs and may create outputs intended for your specific use. If those who fork your repositories create pull requests that then execute your workflows without your approval, this could lead to unintended consequences.

The situation could be worse if you use self-hosted runners. If a pull request is allowed to execute a workflow in an unplanned way, processing and access to resources that shouldn’t be allowed can occur.

Be careful that anyone who is new to collaborating on your repository understands the implications of executing the workflows. That’s why the first two options related to first-time contributors focus on that scenario. A similar caution applies for any automation accounts, such as bots.

The options here progress from least restrictive to most restrictive. In the first option, you can require approval for someone who is doing their first pull request and is also “new” to GitHub and may not be aware of the consequences. The use of *new* here implies an account has never contributed before.

In the second option, you can require approval for anyone who is doing their first pull request to your workflow regardless of their length of time on GitHub.

And with the last option, approval is required for anyone doing a pull request.

The default option is a good middle-ground selection for most repositories.

## Workflow Permissions

Finally on this page, there are options for setting the *default* set of permissions allowed to the GITHUB_TOKEN when workflows are run in the repository ([Figure 9-7](#figdefaultGitToken)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0907.png)<br>
**Figure 9-7. Setting default GITHUB_TOKEN permissions for workflows**

GitHub Actions provides a default access key or *token* that can be used by steps in your workflow that need access to your repository. The accesses can be for working with any of the standard items in a repository, such as content (files), issues, pages, deployments, etc. This token is referenced as GITHUB_TOKEN, and it has a broad set of permissions that you can control at a default level of read-only or read-write here. But within any workflow, you can fine-tune the permissions given to the token through the `permissions` clause. I’ll talk more about this token in the next section on security by design. (The GITHUB_TOKEN is also discussed in [Chapter 6](ch06.html#ch06).)

# GITHUB_TOKEN Default

All new repos default to giving read-only access to the GIT⁠HUB_​TOKEN.

Leveraging the settings just noted is a good way to manage who and what can execute particular workflows. But workflow authors and maintainers should not forget about the more generic ways GitHub allows you to protect content. Those can be employed in your repository to protect workflow and action content as well.

One example is restricting the access and ownership of individuals or teams in a repository. At a broad level, that can be done by making the repository private and/or managing who is invited to be a collaborator. At a more granular level, you can use another GitHub construct, the *CODEOWNERS* file.

## The CODEOWNERS File

In GitHub repositories, users with *admin* or *owner* permissions can set up a CODEOWNERS file in a repository. The purpose of this file is to define individuals or teams that are responsible for (*own*) code in a repository. Example syntax for a CODEOWNERS file is shown in the next listing:

```yml
# Example CODEOWNERS file with syntax info
# Each line consists of a file pattern with owner(s)
# More specific lines further down in file will override earlier

* @global-default-owner   # Global default owner

*.go @github-userid  # Owner for .go files unless overridden later

# tester email is used to identify GitHub user
# corresponding user owns any files in /test/results tree
/test/results/ tester@mycompany.com
```

This file lives in a branch of a repository along with all the other content. Suggested locations in the repository structure are at the root, in a *docs* subdirectory, or in the *.github* subdirectory. The format of the lines is generally a case-sensitive file or directory pattern (similar to the format of a *.gitignore* file) followed by the desired owner on a line. The owners are usually listed by their GitHub user id or team id preceded by @. They must have explicit write access to the repository. For most cases, you can also use their email address that is registered in GitHub.

What the CODEOWNERS file provides is automatic reviewers and approvers for a pull request. When a user opens a pull request for code that fits a pattern in the CODEOWNERS file, the corresponding GitHub user is automatically added as a requested reviewer. Reviewers are *not* automatically requested for draft pull requests.

As it specifically relates to workflow files, an entry can be added for the *.github/workflows* directory in the CODEOWNERS file. (This assumes your workflow files are stored in that directory.) Then any proposed changes to these files will require approval from a designated reviewer.

You can learn more about creating and using a CODEOWNERS file in the [GitHub documentation](https://oreil.ly/RoKXH).

Beyond approval for changes in a branch, you can further restrict access for certain destructive actions, and set requirements for any pushes or changes to tags, by utilizing other control mechanisms available in GitHub, including protected tags, protected branches, and repository rules.

## Protected Tags

Within a repository, you can configure rules to keep contributors from creating or deleting tags. This means that, in order to create protected tags, users have to have *admin* or *maintain* permissions or have a custom role with *edit repository rules* permission in the repository. Likewise, to delete a protected tag, a user must have *admin* permission or a custom role with *edit repository rules* permission.

# Beta Feature

As of the time of this writing, tag protection rules are a beta feature and subject to change.

Creating tag protection rules is a simple process that just involves going to the *Settings* tab, then selecting *Code and automation*, then *Tags*, and finally *New rule*. You’ll then be presented with a dialog box where you can enter the *Tag name pattern* using basic pattern matching syntax. See [Figure 9-8](#creating-a-new-rule-f) for an example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0908.png)<br>
**Figure 9-8. Creating a new rule for a protected tag**

Branches can also be protected in a similar way. But the variety of operations that can affect branches requires more extensive options and rule specifications.

## Protected Branches

For some branches in your repository and the workflows defined in them, you may want to protect them from potentially destructive operations. Such operations could include being deleted or having forced pushes made to them. Branch protection rules in GitHub allow you to create a *gate* where certain conditions must be met in order for operations to proceed. Given that your GitHub Actions workflows can be critical to processes like CI/CD, you may want to add an extra layer of protection. This is especially true for significant branches, such as ones with production workflows, where tighter controls are needed. The list that follows shows some examples of the types of rules that you can create:

* [Require pull request reviews before merging](https://oreil.ly/5EyKq)
* [Require status checks before merging](https://oreil.ly/gdk05)
* [Require conversation resolution before merging](https://oreil.ly/-Qi6Y)
* [Require signed commits](https://oreil.ly/psRbu)
* [Require linear history](https://oreil.ly/Lvf-S)
* [Require merge queue](https://oreil.ly/fe6qE)
* [Require deployments to succeed before merging](https://oreil.ly/89GYt)
* [Do not allow bypassing the above settings](https://oreil.ly/8dKAA)
* [Restrict who can push to matching branches](https://oreil.ly/-dxh3)
* [Allow force pushes](https://oreil.ly/SiORY)
* [Allow deletions](https://oreil.ly/FyicA)

To create a new branch protection rule, you navigate to the *Settings* for the repository, and then in the *Code and automation* section, click *Branches*. On the page that comes up, select *Add branch protection rule* and then fill in the various fields ([Figure 9-9](#branch_protection)). The GitHub [online documentation](https://oreil.ly/frQ1A) has details about what the various fields and settings mean.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0909.png)<br>
**Figure 9-9. Section for branch protection settings**

If you find yourself defining multiple protection rules that you would like to be able to manage as a unit and surface more easily, you may want to look at leveraging repository rules.

## Repository Rules

Within the repository rules framework, a ruleset is a list of rules, identified by a name, that applies to a repository. Creating rulesets allows you to control how users can interact with designated tags and branches in a repository. The point of creating rulesets is to manage, via a collection of rules, who can do certain operations like push commits to a particular branch or delete/rename a tag.

For any ruleset you create, you can specify the following:

* A name for the ruleset
* Which branches or tags the ruleset is using via *[fnmatch](https://oreil.ly/2tugG)* syntax
* Users allowed to bypass the ruleset (if any)
* Which protection rules you want the ruleset to enforce

Creating a new ruleset involves going to the *Settings* tab, then selecting *Code and automation*, then *Rules*, *Rulesets*, and finally *New branch ruleset*. See [Figure 9-10](#creating-a-new-rulese).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0910.png)<br>
**Figure 9-10. Creating a new ruleset**

You may be wondering how rulesets fit in with tag protection and branch protection rules if those are also present in a repository. The set of rules currently allowed in use for rulesets are similar to the standard branch protection rules. They include:

* [Restrict creations](https://oreil.ly/BUQhf)
* [Restrict updates](https://oreil.ly/Lyihq)
* [Restrict deletes](https://oreil.ly/tlxyT)
* [Require linear history](https://oreil.ly/Y2zDo)
* [Require deployments to succeed before merging](https://oreil.ly/AvK7Z)
* [Require signed commits](https://oreil.ly/cw6eF)
* [Require a pull request before merging](https://oreil.ly/_ZVwy)
* [Require status checks to pass before merging](https://oreil.ly/fkU7f)
* [Block force pushes](https://oreil.ly/y6m8R)

So you can start using rulesets without overriding any of your existing protection rules. But rulesets have a few advantages over other kinds of protection rules:

* Multiple rulesets can apply at the same time through a process called *layering* (see sidebar).
* Rulesets have an *enforcement status* that can be set to *Enabled* or *Disabled,* allowing you to easily manage which rulesets are enforced for the repository*.*
* Users with read access can view the rulesets that are active for the repository so they can get more information if they violate a rule. (This is also useful for auditing purposes.)
* Additional advanced functionality for rulesets is available for GitHub Enterprise organizations. More information can be found in [the documentation](https://oreil.ly/5HjLX).

##### Rule Layering

If there are multiple rulesets that apply to the same tag or branch, the rules will be aggregated (as opposed to prioritized), and all rules apply. If there are multiple rules defined in different ways, then the most restrictive rule applies. Layering also takes into account individual branch or tag protection rules.

# Dealing with Protection After a Commit

If the repository has protections like those just discussed in place and a contributor tries to update a branch or tag with a commit that gets blocked by these protections, they will see an error message telling them what was incorrect. However, since commits are immutable in Git, they may need to rewrite their commit history (through a rebase, for example) before being able to push their commits into the repository.

While configuration settings can help protect the code in the repository from unwanted changes and/or execution, they are only one part of ensuring security for your workflows and related pieces. It is also important to deliberately design the functioning of your workflows to be secure from the start. That involves securing important data and anticipating ways that the workflows might be misused. To prevent that, you must think about *security by design*.

# Security by Design

Configuration measures, such as the ones just discussed, can help limit accidental or malicious attack vectors in the environment before you create or execute your workflows. But in case someone, or something, is able to get through those defenses, your workflows need to use good practices to secure access and prevent misuse when run. In this section, I’ll cover two key areas to help with this:

* Securing private data through using secrets and tokens
* Preventing common attacks such as script injection

## Secrets

A *secret* in the context of software refers to a privileged credential stored securely as an object in the system. It acts as a key to unlock sensitive information or protected resources. In GitHub Actions, this sensitive information is often an access token. This token may be needed to allow GitHub or actions to have permissions to do designated operations when a workflow is running. Secrets are encrypted from the time you create them. But they are only decrypted when you use them in a workflow. You can think of them as being like encrypted environment variables.

As with most things in GitHub, you can create secrets at the organization or the repository level. But you can also create secrets at the *deployment environment* level. (Deployment environments are discussed in [Chapter 6](ch06.html#ch06).) With a secret in a deployment environment (versus just stored with the repository), approval must be granted from a specified, required approver before a workflow job can access the secret.

Secret names must be unique at each level—within an organization, a repository, or a deployment environment. The basic rules for secret names are as follows:

* They can only contain alphanumeric characters or underscores and no spaces.
* The GITHUB_ prefix is reserved and cannot be used when you create secrets.
* They must not start with numbers.
* They are not case-sensitive.

# Order of Precedence

If you happen to have secrets with the same name at any combination of the organization, the repository, and/or the deployment environment, secrets will take precedence in this order:

1. Deployment environment
2. Repository
3. Organization

Instructions for creating secrets were provided in [Chapter 6](ch06.html#ch06) and can also be found in [the GitHub documentation](https://oreil.ly/faQCF).

Accessing secrets in your workflow file can be done either by setting them as an environment variable or by specifying them as an input. See the next listing for an example:

```yml
  steps:
    - name: My custom action
      with:  # input secret
        my_secret: ${{ secrets.MySecret }}
      env:  # environment variable
        my_secret: ${{ secrets.MySecret }}
```

Once created, you must also take steps to keep your secrets secure.

## Securing Secrets

The golden rule when dealing with secrets is that “secrets should stay secret.” By this I mean that you should take precautions to prevent exposing the contents of a secret. The precautions are fairly simple but do require some diligence on your part.

The first precaution involves limiting the privileges of any credentials provided by secrets. Keep in mind that any user that has write access to your repository also has read access to all secrets in your repository. A corollary of this is obviously limiting who has write access to your repository.

Even if someone doesn’t have direct read access to your secret, if the secret is exposed in the logs or in some other way, the data can also be exposed to others. So you need to take precautions when printing secrets. GitHub Actions redacts secrets when writing out logs. But to do this, it largely relies on finding an exact match in name and format for the secret value. So, you must be sure to register all secrets used within workflows so the redaction process can find them and be enabled for them.

You also need to avoid using structured data within the contents of the secret, such as YAML, JSON, XML, etc. If you use such structured data in the secret, the redaction algorithm can fail because it can’t match/parse the contents.

Of course, you should not print out secrets yourself as part of your workflow. But you also need to audit your source periodically to make sure secrets are being managed appropriately and not being shared with other systems. And it’s a good idea to review logs to make sure secrets are being redacted as expected.

Another best practice is to establish a regular cycle to audit and rotate secrets. The purpose of the audit here is to review the secrets you have in place and remove any unneeded secrets to prevent any accidental exposures. Rotating secrets means changing their values periodically. The strategy here is that by regularly changing values, you reduce the amount of time that a compromised secret is valid.

Finally, for secrets that are part of a repository environment (as discussed earlier), you can require review for access to them. This can be done via adding *required reviewers* in GitHub.

# Risks with Using Self-Hosted Runners

As opposed to GitHub-hosted runners, self-hosted runners are not guaranteed to be a clean, newly created environment. That may mean that it is possible for subsequent jobs and workflows to read the data left behind by previous ones if precautions are not taken to do proper cleanup.

# Accessing Audit Logs

If you are an admin/owner for an organization in GitHub, you can review the audit log for your organization to see activities executed by members of the organization. For more information, see the [documentation on GitHub](https://oreil.ly/tRR_6).

While secrets allow you to hide/store values securely, they do not have any additional meaning or context to how they are used, what they allow access to, etc. When you need to have a security setting with more context and defined scopes, that’s where tokens come in.

## Tokens

A *token* is an electronic key that can be used to access resources. Tokens are cryptographically generated strings of characters that can be used in place of authentication methods like passwords to provide authentication for accessing resources over network protocols, API calls, etc. Unlike more traditional approaches of authentication, tokens provide several advantages:

* They can easily be stored and referenced programmatically.
* They can be set to have a limited lifetime.
* They can be created for accesses to targeted resources.
* They can have custom permissions and scopes in terms of how much they can access.
* They can easily be created and revoked.

There are two types of tokens you generally use with GitHub Actions: the personal access token (PAT) and the GitHub Token.

### Personal Access Token

If you’ve pushed content to GitHub anytime within the last few years using *https* authentication, you’ve used a PAT.  Several years ago, GitHub replaced its use of passwords with the more secure PAT. As the name implies, this token is for personal access to your GitHub repositories. It is created through your developer settings in GitHub ([Figure 9-11](#defining-a-new-person)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0911.png)<br>
**Figure 9-11. Defining a new personal access token**

# Accessing Your Token

If you are new to personal access tokens, you should be aware that when you create one, that is the only time you’ll be able to see it in clear text ([Figure 9-12](#only-chance-to-view-t)). So you need to make sure to copy it and store it securely for future use. Also, make sure to keep your token secure, just as you would your password.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0912.png)<br>
**Figure 9-12. Only chance to view the actual token**

### Accessing personal access tokens in workflows

Including a token as plain text is never secure. So to access it within a GitHub Actions workflow, you need to first store it in a secret, as shown in [Figure 9-13](#storing-a-token-in-a). (Steps for creating a secret are discussed in [Chapter 6](ch06.html#ch06).)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0913.png)<br>
**Figure 9-13. Storing a token in a secret**

Once you have created a secret to store the PAT, you can access it in your workflow through the *secrets context*, which contains the names and values of secrets stored in GitHub that are available to your workflow runs. To get the value out of any particular secret, you simply access it in your workflow as follows:

```yml
${{ secrets.SECRET_NAME }}
```

So the way to access the example just shown would be this:

```yml
${{ secrets.WORKFLOW_PAT }}
```

# Contexts

A *context* in GitHub Actions is a collection of related variables or properties that can be accessed through a common high-level reference. Contexts are discussed in [Chapter 6](ch06.html#ch06).

Here’s an example of a portion of code where the PAT is passed via the secret as part of a `curl` command to call a GitHub API:

```yml
  steps:
    - name: invoke GitHub API
      run: >
        curl -X POST
        -H "authorization: Bearer ${{ secrets.PIPELINE_USE }}"
```

While a secret with your PAT allows the workflow to perform certain operations on your behalf, GitHub separately needs some basic permissions to do operations with/for your workflows. This is accomplished by a token that GitHub automatically creates for you. This is the subject of the next section.

### The GitHub token

When GitHub Actions are enabled in a repository, GitHub installs a GitHub App in the repository. (See the sidebar for more on what a GitHub App is.) This app has an access token with permissions to your repository. It is commonly referred to as the *GitHub token*. Like the PAT, the token is stored in a secret.

# GitHub Apps

GitHub Apps are hosted applications that can interact directly with the GitHub APIs to add additional functionality to the way you work. As opposed to GitHub Actions, they can act as their own entity or as a specific user and can manage integrations at a level above/across repositories because they use the API.

Prior to executing a job, GitHub gets this token and uses it to execute the job. Since the token’s permissions are limited to the ones specified for your repository or within your workflow, you have controls available for what can be done with the token.

# Token Lifetime

The token for a job expires when the job is completed or after 24 hours, whichever comes first.

### Using the GitHub token in workflows

There are two ways to access the GitHub token and use it in your workflows. You can access it via a *built-in* secret or from the *github* context. Both are valid approaches. Which approach you choose depends on the use case in your workflow.

The first use case is calling an action that consumes the token. Notice the following example (taken from the documentation for a push action). When the token is passed as a parameter to the action, it is accessed as `secrets.GITHUB_TOKEN`. `GITHUB_TOKEN` here refers to a secret that GitHub Actions automatically creates that contains the token. It can then be accessed via the *secrets* context:

```yml
  - name: Push changes
    uses: ad-m/github-push-action@master
    with:
      github_token: ${{ secrets.GITHUB_TOKEN }}
      branch: ${{ github.ref }}
```

Alternatively, the token is available directly via the *github* context. The next example shows how to set an environment variable with the token value from the github context:

```yml
  - name: Create Release
    id: create_release
    uses: actions/create-release@latest
    env:
      GITHUB_TOKEN: ${{ github.token }}
```

# Difference Between Token Access Methods

You might be wondering what the difference is between accessing the GitHub token via the secrets context or the github context. The answer is that they are functionally the same. The secrets context is a more formal way to explicitly show/pass in the token to an action. But even if you do not pass in `secrets.GITHUB_TOKEN` to the `with` clause defined in *action.yaml*, an action can still access the token via the github context.

The GitHub token has an inherent set of permissions based on the default setting for the type of account you’re using in GitHub—enterprise, organization, or individual repository. If those are too restrictive, they can be changed at different levels. The permissions that the token has in any particular repository take effect in the order shown next, with the top one being the default and each one after that able to override the other:

* Permissions as set by default for enterprise, organization, or repository
* Configuration globally in a workflow
* Configuration in a job
* Adjusted to read-only if:
  + Workflow triggered by a pull request from a forked repository
  + Setting is not selected

# Restricting Permissions

As a best practice around security, the GitHub token should only have the minimum permissions needed.

If you need to modify the permissions for the GitHub token, you can use the `permissions` key. As first described in [Chapter 6](ch06.html#ch06), the key can be used in the workflow globally as a top-level key or added only to specific jobs where needed. From the [GitHub documentation](https://oreil.ly/nqlh6), here are the available *scopes* and access values:

```yml
permissions:
  actions: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: read|write|none
  issues: read|write|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none
  repository-projects: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```

As a security measure, if you specify the access for any scope(s), the other scopes that aren’t included are set to `none`. With the caveat that your token should have the minimum amount of privileges needed for your job or workflow, you can set the read or write access for all available scopes via:

```yml
 permissions: read-all|write-all
```

Conversely, you can disable permissions for all available scopes via:

```yml
permissions: {}
```

# Permissions Key and Forked Repos

Read permissions can be added/deleted for forked repositories via the permissions key. But write access can’t be granted unless an admin user has selected `Send write tokens to workflows from pull requests` as an option in the Actions settings.

An example of adding additional permissions to the default permissions of the GitHub token at the level of the overall workflow is shown in the last lines of this code snippet:

```yml
name: Java CI with Gradle

on:
  push:
    branches: [ "blue", "green" ]
  pull_request:
    branches: [ "main" ]
    types:
      - closed
  workflow_dispatch:
    inputs:
      myVersion:
        description: 'Input Version'
      myValues:
        description: 'Input Values'

permissions:
  contents: write
```

If you want to see what permissions the GitHub token has in your workflow, an easy way to do that is to look at a run of your workflow and expand the GITHUB_TOKEN section in the output. [Figure 9-14](#seeing-the-permission) shows an example of a run with the code snippet just shown.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0914.png)<br>
**Figure 9-14. Seeing the permissions of the GitHub token in a workflow run**

One last note on the interaction between the GitHub token and your workflows is worth mentioning. Since the GitHub token may have permissions to perform tasks within your workflow, in theory it could spin up new workflow runs. In the worst case, this could cause workflow runs to run recursively.

To avoid this, when you use the repo’s token to do tasks, GitHub prevents events triggered through use of the token from causing a new workflow run, with two exceptions. The `workflow_dispatch` and `repository_dispatch` events can still create new workflow runs. This makes sense from the point of view of having workflows able to trigger other workflows when intended. I’ll talk more about this pattern in [Chapter 12](ch12.html#ch12) on advanced workflows.

Using secrets and tokens is a key design principle to protect sensitive data and access when a workflow is doing its intended processing. But it is also important to design against malicious data, which can cause unintended processing, being *fed* into a workflow. I’ll discuss that general problem and show an example of a use case next.

## Dealing with Untrusted Input

When an event triggers your workflow, that trigger brings with it a set of information related to the event. This information includes standard data associated with operations in GitHub like the SHA value of the change, commit message, pull request data, author info, etc. This data provides a context for the workflow to reference as it’s running—the GitHub context. (Contexts are discussed more in [Chapter 6](ch06.html#ch06).)

The set of data points provided with the GitHub context is long. As of the time of this writing, you can see [the complete list of data](https://oreil.ly/IqslF). From a security perspective, though, we have two broad classifications of it: *generally trusted* and *generally untrusted*.

The *generally* terminology is here because there are no guarantees, either that data can be manipulated or not. But data that is more permanent (such as the repository name) or generated via GitHub (such as SHA values or pull request numbers) is less likely to be exploited—at least through these kinds of attacks.

On the other hand, there is a large amount of data available through the GitHub context that is tied to the current event and/or has user-configurable information in it and should not be trusted. Here are some examples:

* Issue titles and bodies
* Pull requests titles and bodies
* Review bodies and comments
* Commit messages
* Author emails and names
* Pull request references and labels

# Hacking and Validation

You might be inclined to think that items like email addresses can’t be hacked if proper validation checking is in place. But besides the usual alphanumeric characters, the part of the email address before the @ can include a number of other printable characters, including “!#$%&'*+-/=?^_`{|}~”. Anyone familiar with shell programming can probably start to see how this could be used with commands like `echo` to gather data.

Consider the case where an attacker was able to insert actual code/shell commands into this seemingly benign data. If, in your workflow/action (or ones that you are pulling in to use), that data gets passed through to API calls or the shell on a runner, then the code could be interpreted and executed there.

# Self-Hosted Runner Security Measures

While this chapter doesn’t go into all of the details about the security differences between GitHub-hosted runners and self-hosted runners, there is one basic difference. Self-hosted runners, by default, persist the same runner across multiple jobs, whereas GitHub-hosted runners spin up a new runner for each job.

You can make self-hosted runners only run one job by defining them as *ephemeral.* This can be done with an option on the config script run at setup time or via a REST API call. [Chapter 5](ch05.html#ch05) on runners has more details on this functionality, also called *just-in-time runners*.

[Chapter 5](ch05.html#ch05) also has a reference to where to find another security asset—an SBOM for the image releases used by GitHub-hosted runners.

There is [an excellent article](https://oreil.ly/WPxQK) on this in GitHub, so I won’t repeat all of the details it provides. However, in the next section, I’ll provide an example of a common, untrusted data vulnerability known as *script injection*, show how it can be exploited with a workflow, and then discuss how you can guard against that.

### Script injection

*Script injection* refers to a security vulnerability whereby an attacker can inject malicious code into user input, such as a text field on a website. Since GitHub Actions functions in a browser, it’s important to consider whether exposed data like inputs you define may be susceptible to these kinds of attacks. And it’s even more important to consider what can be done to prevent your workflows from being exploited.

When your workflow runs and asks for input, those values ultimately end up being passed, as data, to the workflow executing on the runner. And, if you’re not guarding against it, code injected in those input strings can end up executing on the runner as instructions.

The process starts when an event happens in GitHub that triggers your workflow. That triggering also provides useful context data about the event. That data can then be accessed in your workflow code. Included is basic info about the origin of the event itself, such as the user that made the change, the branch name, etc. This information is passed on to the workflow as part of the official GitHub Actions *github context*.

But with that context, you also have a lot of data passed through that can be manipulated by an attacker. This is especially true for data values that originate from human input, such as the bodies, title, and comments of GitHub issues and pull requests. Even email addresses and names of authors on commits are susceptible. These are the kinds of items that you should treat as untrusted input.

As an example, consider a very simple workflow script that just prints out the commit message of a push that triggered it. The code might look like the following:

```yml
name: sidemo

on:
  push:
    branches: [ "main" ]

jobs:
  process:
    runs-on: ubuntu-latest

    steps:
      - run: echo ${{ github.event.head_commit.message }}
```

Now suppose someone makes a push and puts in a commit message like this:

```yml
`echo my content > demo.txt; ls -la; printenv;`
```

Oddly enough, that is a perfectly valid commit message. In that case, the job would run and execute the commands embedded in the commit message—creating a file, getting a directory listing, and printing out the environment on the runner. [Figure 9-15](#example-output-from-d) shows what output from that would look like. Note the directory listing that follows the `printenv`.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0915.png)<br>
**Figure 9-15. Example output from demo code**

This is obviously a contrived example, but you get the idea. This is only one example of a way that untrusted input can get you into trouble. Even items that you would think should be secure can be risky. For example, suppose I create a new secret ([Figure 9-16](#creating-new-secret)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0916.png)<br>
**Figure 9-16. Creating a new secret**

And then code is added to the workflow to print it out:

```yml
- run: echo ${{ secrets.SIDEMO_SECRET }}
```

When this code is executed, the secret info is redacted, as it should be ([Figure 9-17](#redacted-secret-data)). This is part of the built-in functionality of GitHub Actions for secret management.

However, what if I update the secret to have a different kind of data in it? (See [Figure 9-18](#storing-code-in-secre).)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0917.png)<br>
**Figure 9-17. Redacted secret data**

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0918.png)<br>
**Figure 9-18. Storing code in secret**

The data in this case is a set of code as shown in the following listing:

```yml
printf "Starting custom code\n"
mkdir foo
echo data.txt > foo/foo1.txt
echo more > foo/foo2.txt
ls -la foo
rm -rf foo
ls -la
printf "Executed!\n"
```

When I run this job, although the data in the secret is redacted, note that the code has actually been executed! (See [Figure 9-19](#code-executed-from-se).)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0919.png)<br>
**Figure 9-19. Code executed from secret**

So how do you fix this? That’s the topic of the next section.

### Preventing script injection vulnerabilities

The issues related to shell command injection happen because of the way the strings are interpreted on the runner:

* The run command executes within a temporary shell script on the runner.
* Before this temporary shell script is run, the expressions inside `${{ }}` are evaluated.
* Then, substitution happens with the resulting values from the evaluation.

To prevent and mitigate the exposures of script injections, there are a couple of strategies that you can employ.

A good approach is to avoid using inline scripts and call an action, if one is available, to do the same operation. This mitigates issues because context values are passed to the action as an argument instead of being directly evaluated.

Alternatively, if you need to call the `run` command as part of your workflow, then the best practice is to capture any values you pass to the `run` command in an intermediate variable. That way the value is passed as an environment variable instead of directly being evaluated and executed.

For example, to fix the initial use case I showed, you could change the code in your workflow to this:

```yml
    steps:
      - env:
          DATA_VALUE: ${{ github.event.head_commit.message }}
        run: echo $DATA_VALUE
```

Using the same example input that was used before, this code does the simple echoing of the data instead of executing it ([Figure 9-20](#expected-output-from)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0920.png)<br>
**Figure 9-20. Expected output from running with an environment variable**

The code that was executing the contents of the secret can be changed in the same way:

```yml
    steps:
      - env:
          DATA_VALUE: ${{ secrets.SIDEMO_SECRET }}
        run: echo $DATA_VALUE
```

When run, the secret info is not executed ([Figure 9-21](#expected-output-from2)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0921.png)<br>
**Figure 9-21. Expected output from running with the secret data in an environment variable**

Even when you have designed your workflow to secure sensitive data, prevented untrusted input, and done what you can in your implementation to prevent exposures, there is still a weakest link—your dependencies. To make your workflows as bullet-proof as possible, there are best practices you need to follow in your design as it pulls in other actions, workflows, or third-party components.

## Securing Your Dependencies

You no doubt already understand the importance of making sure the third-party pieces you use in your product, your code, or your infrastructure are as secure as possible. In GitHub Actions, the same holds true for any additional actions or workflows that you allow to interact with yours.

I’m using the term *allow* purposely here to point out that when you use a third-party action or workflow, you are allowing them certain access to your code and the systems it executes on. When you reference an action with the `uses` directive, you are running third-party code and providing access to your computing time, computing resources (if using a self-hosted runner), secrets used in the same job, and your GitHub repository token. You’ve opened the door and invited them in. Now they have lots of opportunities to take and do what they want.

The ways to guard against this are items previously talked about or are likely common sense as you think about this. I’ll list several here as reminders, though:

* Use the principle of least privilege. Your code should run with the least privileges necessary to do any particular task.

  + This applies to secrets you create and use as well as the GITHUB_TOKEN.
  + In the case of the token, you can set the permissions to be more limited by default and then update them in the workflow with the `permissions` clause as described previously.

* Verify actions you use. Since any action you use has access to secure operations and information, it’s important for you to review them prior to incorporating them.
  + At a minimum, you can look for GitHub’s *verified creator* badge to know that GitHub has done some minimal verification on these actions.
  + If you are targeting integration with a particular industry product or company, you can also look for an action supplied by that company.
  + Some actions will also have *stars* attached to them to indicate a sort of rating in terms of how many users have actually taken the time to add a star. (More stars may indicate a more heavily used action.)
* Review the code. Note that none of the previous items imply that the action is secure, though.
  + It is up to you to review/audit the code just as you would (hopefully) do for any third-party pieces you choose to use in other software.
* Use the best reference. By reference, I mean the Git reference to the version of the workflow code that you pull in. You have a number of options in the way you do this.
  + Branch name: `uses: creator/action-name@main`, for example. This approach will always use the latest version from the branch. As such, you will get the leading edge of the code but also assume more risk for picking up any incomplete or breaking changes.
  + Tag/release: `uses: creator/action-name@v#`, for example. This is a more common way to access actions. You’re not necessarily getting the latest, just whichever version is tagged with this one at this point in time. This tag will likely be moved over time as new minor versions are created. (Note that GitHub prioritizes creators who use semantically versioned actions.)
  + Full changeset hash: `uses: creator/action-name@64004bd08936bec272​60​53​ded6d09d33290ef437`, for example. This is the most explicit and safest way to reference a particular version of an action.
  + Reference your own copy by forking the version of the action you want. This is secure, but you also have to plan for how you want to pick up updates, including bug fixes and security fixes.

# Referencing by Short Changeset Hash

References such as `uses: creator/action-name@64004bd` were allowed at one point but are not any longer. The reason has to do with a particular kind of attack that could be done with them to prevent any workflows referencing actions in this manner from running.

Paying attention to configuration in your GitHub environment and good design practices will go a long way towards having secure workflows. However, the responsibility for ensuring security does not end with those activities. Once the workflows are live, there must be diligence and awareness of what changes are being made to them or that might affect them. This can be managed best through using good monitoring practices as described in the last part of this chapter.

# Security by Monitoring

GitHub is, by design, a collaborative environment. In most cases, this means that there are intentional ways for code in your repository to be modified by others (such as pull requests) and unintentional ways (such as someone modifying the tagged version of an action or third-party component you use). And the collaborative model means that it’s easier for changes to be introduced frequently and quickly.

This approach is great for collaboration. But it presents additional risks for securing your workflows and actions. Those ways of introducing changes can occur throughout the lifecycle of your repository. And they can be introduced long after you have done all the necessary configuration and design to try and make your workflows secure. As I’ll show, it can be easy to have code masquerade as valid (or even intended as valid) that ultimately results in problems or opens you up to attacks.

The best defenses against these sorts of issues involves due diligence for actively scanning, reviewing, and safely validating incoming changes with secure pull request processes. You can broadly categorize the set of these activities as *monitoring.* The next few sections will look at each of these areas.

## Scanning

GitHub Actions makes it easy to set up code scanning for your repositories via its starter workflows. If you go to the section for starter workflows and select the *Security* category, you’ll see a number of options you can select from to help scan your code for vulnerabilities ([Figure 9-22](#starter-actions-for-s)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0922.png)<br>
**Figure 9-22. Starter actions for scanning**

The idea is that these workflows can be added to your repository and then can run independently to scan the code in the repository. This may be as easy as just clicking the Configure button and then doing any optional changes to update the code type, scanning intervals, and so on. An example screen from the initial configure selection for the CodeQL action is shown in [Figure 9-23](#initial-scanning-acti).

![ch](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0923.png)<br>
**Figure 9-23. Initial scanning action configuration**

After being set up as a workflow, the scanning will trigger based on the events defined in the `on` section—in this case, a `push`, `pull request`, or `schedule`. After a run, you can look at the results in the job execution ([Figure 9-24](#codeql-run-for-a-repo)).

Additionally, GitHub has automated scanning functionality called *Dependabot* to check dependencies for updates and security issues. In terms of workflows, Dependabot vulnerability checking can be set up to ensure that references to actions that you use in your workflow files are kept current. For each reference to an action, Dependabot will check to see if there is an updated version available. If there is, it will send you a pull request to update the workflow to reference the latest version. For more details on how to set this up, see the [documentation](https://oreil.ly/LrMCT).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0924.png)<br>
**Figure 9-24. CodeQL run for a repository**

# Additional Code Scanning Options

Dependabot is only one source of scanning that you can set up for your repositories in GitHub. If you click the *Security* tab in your repository and look under the Vulnerability Alerts section on the left, there is a *Code Scanning* item you can click to set up additional tools/scanning actions to provide reports and alerts from scanning code in your repository.

# OSSF Scorecard for Actions

An additional tool for security scanning is the [OSSF Scorecard](https://oreil.ly/4pQJ-). This tool reviews a set of factors related to security in software repositories and provides a score for each area from 1 to 10.

For GitHub projects, the [Scorecard GitHub Action](https://oreil.ly/0FqJt) can be used to get the scores. The action runs anytime there is a change to the repository and creates alerts that can be reviewed in the repository’s Security tab. It can also provide badges (in the same form as the status badges) for repositories to easily show the scores. Installation instructions can be found [in the documentation](https://oreil.ly/g-vaO).

As of the time of this writing, GitHub is planning to utilize the scorecard on future actions put into the Actions Marketplace. The exact set of criteria that will be standardly run and reported for each action is TBD, but the expectation is that action creators/owners would need to address any *High* vulnerabilities quickly.

While scanning can catch vulnerabilities of certain types that have been introduced since the last scan, it is not a substitute for standard best practices, such as code review, when you have changes being requested or added. As discussed earlier in the configuration section, a good first step is to set up a CODEOWNERS file so that there is clear review and approval responsibility for code in a repository.

GitHub provides extensive code review functionality built in through its interface. This is especially useful when managing pull requests. But, ironically, when working with workflows, pull requests, via their automation, can also be one of the most vulnerable mechanisms for security. So you need to take precautions.

## Processing Pull Requests Securely

When set to trigger on a pull request event, a workflow can test and prove the validity of the code in the requested changes before they are merged. This sort of *pre-flight/pre-merge* check can be very useful in identifying broken or bad code and preventing it from being merged. This follows one of the key principles of automated CI/CD.

However, if not done carefully, with GitHub Actions, this type of flow can also provide a large attack surface on your existing code base, runner systems, etc. If a workflow is designed to be triggered on a pull request, it often is designed to run some sort of build and test processing against the code in the pull request. This leaves open any number of attack vectors through the code being introduced and run, including the following:

* Modifying build scripts
* Modifying test cases/suites
* Leveraging any kind of pre-/post-processing built into the tooling used by the repository
  + An example here could include slipping in a malicious package for a package manager install of prerequisites

I’m sure you can start to imagine any number of ways your code base could be exploited with these kind of scenarios. Fortunately, GitHub also realized the potential impacts here. As a result, the standard `pull_request` trigger for a workflow mitigates the risk by doing the following:

* Preventing write permissions to the target repository for a pull request
* Preventing access to the repository’s secrets from an external fork
  + Access is allowed for pull requests originating from a branch in the same repository

These are good safeguards for the vast majority of cases. But, once in a while, you may find yourself needing that write permission, or access to secrets, to fully vet the content of a pull request. For those situations, GitHub has provided the `pull_request_target` trigger. This trigger runs in the context of the target repository of the pull request, as opposed to running in the context of the merge commit. It allows for more automated exercising/review of the pull request. It can also allow workflows to perform operations like adding comments on a pull request or labeling them so they can be automatically categorized or flagged for further review.

# Pull Requests from Forks

To be clear, when it comes to the pull requests discussed in conjunction with the `pull_request_target` trigger, this refers to pull request from *forks* of the repository, not pull requests from other branches in the same repository. As noted, pull requests triggered from a branch in the same repository already have write permissions and access to secrets.

But what happens if the `pull_request_target` is triggered and someone has slipped in malicious code? The code in the pull request could now have access to the secrets and write permissions in the target repository. GitHub thought ahead about that and added a safeguard; the event triggered by `pull_request_target` doesn’t execute anything from the workflows in the pull request itself. It just executes the workflow code and configuration already in the base repository; the existing workflows in the target that have presumably already been run and are known to be safe.

Great—so you should be safe, right? In most cases, yes—unless you do something in your workflow code that circumvents the safeguards. The most common and easiest example is using actions/checkout within your workflow to check out the code from the pull request’s repository’s HEAD.

Consider the following code:

```yml
name: some action
on: [push, pull_request, pull_request_target]

jobs:
  pr-validate:
    name: Validate PR
    runs-on: ubuntu-latest
    steps:
      ...
      - name: Checkout Repository PR
        if: ${{ github.event_name == 'pull_request_target' }}
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
```

Because this code has an execution path based on a `pull_request_target`, when that path is executed, the workflow is given access to secrets and a full read and write GitHub token. Then the checkout path puts the code from the pull request repository onto the runner system, leaving the configuration vulnerable to exploitation.

The main pathway for exploitation centers around an attacker rewriting some commonly used script, for example, a build script such as a Gradle wrapper file (*gradlew*), or changing the list of third-party pieces that get installed, for example, modifying a *requirements.txt* file for Python execution.

Another approach can be to change the pre- or post-hook processes that are called. Basically anything that can be used to pull in other code, including *local actions* (*action.yml* files housed in the same repository) that are brought in through a pull request are exposures. (Local actions are discussed in [Chapter 11](ch11.html#ch11).)

To better understand this, see the next section.

## Vulnerabilities with Workflows in Pull Requests

To understand more about how a security vulnerability can happen with a pull request scenario, let’s work through a simple example. Suppose I have a basic repository with a simple Java program and the Gradle build pieces to build it.

The project (*pr-demo*) has the following structure:

```sh
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
  └── main
      └── java
          └── echoMsg.java
```

Here, *src/main/java/echoMsg.java* is my program, and the other pieces are files that Gradle needs to be able to build the project.

You want to automate the CI and build processes for this repo using a GitHub Actions workflow. You can select the *Actions* menu and find a suitable starter workflow to use, such as *Java CI with Gradle*, and then configure it as the workflow *.github/workflows/gradle.yml*. The initial code in *gradle.yml* is shown next:

```yml
name: Java CI with Gradle

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb9
8943c375e1
      with:
        arguments: build
```

After an initial commit, this code will execute and build the Java source code.

Also, for future work, I’ll go ahead and generate a personal access token and add it as a secret named *PAT*.

Suppose someone else comes along and forks this repository to make some changes. They then add some code to gather debug information and automatically report an issue if there is a failure. The added code is shown here:

```yml
permissions:
  ...
  issues: write

    - name: Get Debug Info
      run: |
        echo "DEBUG_VALUES=$(git
          --work-tree=/home/runner/work/pr-demo/pr-demo config
          --get remote.origin.url)" >> $GITHUB_ENV
        echo "DEBUG_VALUES2=${{ github.workflow }}" >> $GITHUB_ENV

    - name: Create issue using REST API
      if: always() && failure()
      run: |
        curl --request POST \
          --url https:/https://learning.oreilly.com/api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          --header 'content-type: application/json' \
          --data '{
            "title": "PR evaluated successfully",
            "body": "DEBUG_VAL1: ${{ env.DEBUG_VALUES }}
                     DEBUG_VAL2: ${{ env.DEBUG_VALUES2 }}"
            }' \
          --fail
```

I’ll talk more about what this code does in a moment. But assume that the user who forked the code now opens a pull request back to the original repository. GitHub will prompt to approve running workflows for first-time contributors. But nothing about this code looks dangerous, so assume that the approval is granted. The code will run cleanly, and then the merge request can be completed and the code will be merged in. So far so good.

Now, suppose the user who forked the code makes some simple changes, as follows. Can you tell what the differences are, and what the code will now do?

```yml
permissions:
  ...
  issues: write

    - name: Get Debug Info
      run: |
        echo "DEBUG_VALUES=$(git
          --work-tree=/home/runner/work/pr-demo/pr-demo config
          --get http.[token value location])" >> $GITHUB_ENV
        echo "DEBUG_VALUES2=${{ secrets.PAT }}" >> $GITHUB_ENV

    - name: Create issue using REST API
      if: always()
      run: |
        curl --request POST \
          --url https:/https://learning.oreilly.com/api.github.com/repos/${{ github.repository }}/issues \
          --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          --header 'content-type: application/json' \
          --data '{
            "title": "PR evaluated successfully",
            "body": "DEBUG_VAL1: ${{ env.DEBUG_VALUES }}
                     DEBUG_VAL2: ${{ env.DEBUG_VALUES2 }}"
            }' \
          --fail
```

Here are the changes. In the *Get Debug Info* section, the first call to get the URL of the *remote.origin* has been changed to get a configuration value for *http.[token value location]*. (Due to security concerns, I won’t publish the exact location here.) But *[token value location]* references an actual location in the Git config context on the runner system that contains the GITHUB_TOKEN value.

The second call to get a value and put it in the environment is now pulling the value of the PAT secret, which was set up to contain the personal access token for the user.

Finally, the conditionals at the start of the step to create an issue if there’s a failure have been changed. Notice that the *&& failure()* piece has been removed. This means that only the *always()* clause is in effect, so this code will always execute, whether there was a previous failure or not.

If a pull request is submitted based on this code, the initial code check will likely fail in the target repo due to it not having the required permission to create the failure issue there. However, if a repository owner isn’t checking closely enough and decides to go ahead and merge the code, they would end up with a new issue created with contents like the ones shown in [Figure 9-25](#repo-issue-with-expos).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0925.png)<br>
**Figure 9-25. Repo issue with exposed data**

In this case, the exposed data in the *DEBUG_VAL1* field is the GITHUB_TOKEN value. And the exposed data in the *DEBUG_VAL2* field is the personal access token for the user of the target repository!

Now, what would happen if the workflow in the original repository had a workflow trigger for *pull_request_target* instead of just *pull_request*?

```yml
on:
  push:
    branches: [ "main" ]
  pull_request_target:
    branches: [ "main" ]
```

This means that when a pull request is made against the original repository, the pull request checking will be done by running the workflows in the target (original) repository. In this case, the workflows in the original target repository will run successfully and will not exercise the workflow with the code to steal the token and the secret. This is good in that it will not run the malicious code. But it is bad in that it gives a false sense that everything is OK. And that may lead to the maintainer of the target repo merging in the malicious code, based on the target workflows running OK. Moral of the story: *always review any changes in workflows carefully before merging a pull request.*

# Automatic Detection of GitHub Token Exposure

While the GitHub token was able to be exposed in an issue, it should be noted that GitHub runs checks for this sort of exposure and fairly quickly can detect the issue and revoke the token. An example of the email you get in this kind of situation is shown in [Figure 9-26](#github-token-detected).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0926.png)<br>
**Figure 9-26. GitHub token detected in issue and revoked**

So far, I’ve only covered examples of workflows introducing vulnerabilities, but of course any code in a forked repository can potentially introduce attacks.

## Vulnerabilities with Source Code in Pull Requests

When running in a GitHub Actions environment, it’s important to remember that, just as workflows operate on source code, source code can also affect workflows, if engineered to do so.

For example, suppose the *gradlew* wrapper script in the forked repository is modified as shown here:

```sh
case $i in
    (0) set -- ;;
    (1) set -- "$args0" ;;
    (2) set -- "$args0" "$args1" ;;
    (3) set -- "$args0" "$args1" "$args2" ;;
    (4) set -- "$args0" "$args1" "$args2" "$args3" ;;
    (5) set -- "$args0" "$args1" "$args2" "$args3" "$args4" ;;
esac
fi

VALUE1=`git --work-tree=/home/runner/work/pr-demo/pr-demo config --get http.[token value location] | base64`
echo VALUE1=$VALUE1

GIT_REPO=`git --work-tree=/home/runner/work/pr-demo/pr-demo config --get remote.origin.url`
echo GIT_REPO=$GIT_REPO
GIT_USER=`echo $GIT_REPO | cut -d'/' -f4`

if [ "$GIT_USER" != gwstudent ]; then
  echo We have access to the file system!
  for i in `ls -R /home/runner/work`; do
    echo "Deleting $i !"
  done
fi

# Escape application args
save () {
  for i do
    printf %s\\n "$i" | sed "s/'/'\\\\''/g;1s/^/'/;\$s/\$/' \\\\/"
  done
  echo " "
}

APP_ARGS=$(save "$@")

# Collect all arguments for the java command, following shell
eval set -- $DEFAULT_JVM_OPTS $JAVA_OPTS $GRADLE_OPTS "\"-Dorg.gradle.appname=$APP_BASE_NAME\"" -classpath "\"$CLASSPATH\"" org.gradle.wrapper.GradleWrapperMain "$APP_ARGS"

# by default we should be in the correct project dir
cd "$(dirname "$0")"
fi

exec "$JAVACMD" "$@"
```

Did you find the bad code? Take a look at lines 158–170. I’ll break down what these lines are doing for you.

Line 158: Git is used to grab the value of the GitHub token for the repo, as done in the earlier example. But notice that it also pipes it through a simple base64 encoding. I’ll explain why next.

Line 159: The value from the code in line 158 is echoed out here. Normally if you tried to echo out the GitHub token value, you would simply see *** for the token part. By putting this through a base64 encoding, we have something that can be printed out. While it is encoded, it is a simple matter to get back to the actual token value by grabbing the output and running it through a base64 decoding.

Lines 161–162: The Git repo path is grabbed and printed out.

Line 163: The user space part of the repo name is parsed out to use as a check on whether the code that follows should run.

Lines 165–170: If this is not being executed in the originating repo, then this set of code runs. Note that in this demo case, it is only echoing out the deletes instead of actually executing them. By the actual directory used in the example, you may have guessed correctly that this would be doing the deletes on the filesystem of the runner.

When this code is committed and run, the output from the log of the Gradle build that is run via the Java CI workflow looks like [Figure 9-27](#output-of-workflow-ru).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098131067/files/assets/lgha_0927.png)<br>
**Figure 9-27. Output of workflow running Gradle build with modified build file in originating repo**

If you were to create a pull request on this back to the original project, then when the pull request checks were run, you would see the base64-encoded GITHUB_TOKEN value for the target repository *and* the `delete` commands would be executed as shown in the build log listing:

```log
2022-12-03T23:14:01.7666595Z
[command]/home/runner/work/pr-demo/pr-demo/gradlew build
2022-12-03T23:14:01.7761283Z VALUE1=
QVVUSE9SSVpBVElPTjogYmFzaWMgZUMxaFkyTmxjM010ZCc5clpXNDZaMmh6WDFsSlRt
NXBWRzFr VVRFNVFuaFJNakJNY0ZGNGFYSn[********]=
2022-12-03T23:14:01.7772967Z GIT_REPO=
https://github.com/techupskills/pr-demo
2022-12-03T23:14:01.7790669Z We have access to the file system!
2022-12-03T23:14:01.7817919Z Deleting /home/runner/work: !
2022-12-03T23:14:01.7818289Z Deleting _PipelineMapping !
2022-12-03T23:14:01.7819913Z Deleting _actions !
2022-12-03T23:14:01.7820998Z Deleting _temp !
2022-12-03T23:14:01.7869028Z Deleting pr-demo !
2022-12-03T23:14:01.7869518Z Deleting
/home/runner/work/_PipelineMapping: !
2022-12-03T23:14:01.7869909Z Deleting techupskills !
2022-12-03T23:14:01.7870306Z Deleting
/home/runner/work/_PipelineMapping/techupskills: !
2022-12-03T23:14:01.7870722Z Deleting pr-demo !
2022-12-03T23:14:01.7871250Z Deleting
/home/runner/work/_PipelineMapping/techskills/pr-demo: !
2022-12-03T23:14:01.7871680Z Deleting PipelineFolder.json !
```

## Adding a Pull Request Validation Script

Another common approach to validation of pull requests is creating a dedicated workflow in the target repository to validate the content of an incoming pull request. You can set it to be triggered on a pull request but run in the target environment, via the `pull_request_trigger` event. An example of such a script is shown here:

```yml
name: Evaluate PR

on:
  pull_request_target:

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb
      with:
        arguments: build
```

If you repeat the pull request now with this workflow in place, then since this workflow runs in the target environment, the malicious code will not execute:

```yml
2022-12-04T03:26:46.2729823Z
[command]/home/runner/work/pr-demo/pr-demo/gradlew build
2022-12-04T03:26:47.9965751Z
Starting a Gradle Daemon (subsequent builds will be faster)
2022-12-04T03:26:53.4999474Z > Task :compileJava
2022-12-04T03:26:53.5001847Z > Task :processResources NO-SOURCE
2022-12-04T03:26:53.5002919Z > Task :classes
2022-12-04T03:26:53.5968858Z > Task :jar
2022-12-04T03:26:53.5969399Z > Task :assemble
2022-12-04T03:26:53.5970105Z > Task :compileTestJava NO-SOURCE
2022-12-04T03:26:53.5970759Z > Task :processTestResources NO-SOURCE
2022-12-04T03:26:53.5971293Z > Task :testClasses UP-TO-DATE
2022-12-04T03:26:53.5971750Z > Task :test NO-SOURCE
2022-12-04T03:26:53.5972224Z > Task :check UP-TO-DATE
2022-12-04T03:26:53.5972670Z > Task :build
2022-12-04T03:26:53.5972924Z
2022-12-04T03:26:53.5973537Z BUILD SUCCESSFUL in 7s
2022-12-04T03:26:53.5973950Z 2 actionable tasks: 2 executed
2022-12-04T03:26:53.9990279Z Post job cleanup.
2022-12-04T03:26:54.1872827Z Stopping all Gradle daemons
```

# Changing All Workflows

Even though the evaluation workflow only runs on a `pull_request_target` event, other workflows may still run the bad code if they have a `pull_request` target. Be careful of having multiple workflows that have overlap in terms of the events they respond to if that’s not what you intend.

A mistake that users sometimes make when using the `pull_request_target` trigger is to evaluate the source code from the remote project in the target environment. Most commonly, this is done by modifying the checkout step to check out the code from the source of the pull request. In the previous example, the change might look like the following for the *Evaluate PR* workflow:

```yml
name: Evaluate PR

on:
  pull_request_target:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
      with:
        ref: ${{ github.event.pull_request.head.sha }}

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb
      with:
        arguments: build
```

Note the two lines in bold after the *uses:* `actions/checkout@v4` call. The intent here is to check out the code from the pull request’s original repository and run it in the environment of the target repository.

If the same pull request is done again with these changes in the evaluation workflow, you’ll see something like the following in the logs:

```log
2022-12-04T04:47:59.2928413Z
 [command]/home/runner/work/pr-demo/pr-demo/gradlew build
2022-12-04T04:47:59.3013255Z VALUE1=QVVUSE9SSVpBVElPTqojYmFza
WMgZUMxaFkyTmxjM010ZEc5clpXNDZaMmh6WDBKQ1ZVNWtXRTVW
UkV0bWFUaHNhMjlhT1V4ak9XSlFRMWxIUlZnNFZqQ[********]
2022-12-04T04:47:59.3030530Z GIT_REPO=
https://github.com/techupskills/pr-demo
2022-12-04T04:47:59.3043854Z We have access to the file system!
2022-12-04T04:47:59.3072897Z Deleting /home/runner/work: !
2022-12-04T04:47:59.3073506Z Deleting _PipelineMapping !
2022-12-04T04:47:59.3074194Z Deleting _actions !
2022-12-04T04:47:59.3075506Z Deleting _temp !
2022-12-04T04:47:59.3076280Z Deleting pr-demo !
2022-12-04T04:47:59.3076774Z Deleting /home/runner/work/
_PipelineMapping: !
2022-12-04T04:47:59.3077300Z Deleting techupskills !
2022-12-04T04:47:59.3077805Z Deleting /home/runner/work/
_PipelineMapping/techupskills: !
```

Notice that now the bad code has been executed within the context of the target environment! This is shown by the repo path, and also the base64-encoded token value is different.

So how do you mitigate against this?

## Safely Handling Pull Requests

There are a couple of strategies to prevent the kind of issues covered in the last couple of sections.

If a workflow does not need access to the target repository’s secrets and doesn’t need write permissions, use `pull_request` instead of `pull_request_target` so that the operations are not run in, and don’t have access to, the target repository’s environment.

If a workflow does need access to the target repository’s secrets and/or needs write permissions, consider splitting the workflow into multiple pieces. A [GitHub post](https://oreil.ly/H3ZrL) describes the process in more detail, but essentially you split the workflow processing into two parts—something like this:

```yml
name: Workflow 1 Handle untrusted code

# R/O repo access
# Cannot access secrets
on:
  pull_request:

jobs:
  process:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: do processing of pull request securely
        ...
      - name: persist results from processing
        uses: actions/upload-artifact@v4
        with:
          <results of processing>
```

```yml
name: Workflow 2 Do processing that needs r/w access and/or secrets

# R/W repo access
# Access to secrets
on:
  workflow_run:
    workflows: ["Workflow 1 Handle untrusted code"]
    types:
      - completed

jobs:
  process:
    runs-on: ubuntu-latest

    if: >
      github.event.workflow_run.event == 'pull_request' &&
      github.event.workflow_run.conclusion == 'success'
    steps:
      - name: get results from processing securely
        uses: actions/download-artifact@v4
        with:
          <results of processing>
      - name: do processing with results
```

The first workflow is triggered by the `pull_request` event, so it does not have/need write access to the target repository or access to its secrets. It can do whatever processing needs to be done on the candidate change without risking execution in the target environment. In order to make processing results (build logs, test coverage, etc.) available to other workflows, it uploads them as an artifact.

The second workflow leverages the `workflow_run` event. This event was introduced for situations like this to allow running workflows with write permissions and secrets access. It does the inverse of the first workflow, downloading the persisted artifacts, and then doing whatever processing is needed, such as commenting on/updating the actual pull request based on the results.

Another approach noted in the [GitHub security documentation](https://oreil.ly/XYROt) is having someone with responsibility and permission manually review any incoming pull requests and assign a label to them that means it is safe to process in the target environment. It could look something like this if the label being assigned was *allow*:

```yml
on:
  pull_request_target:
    types: [labeled]

jobs:
  processing:
    ...
    if: contains(github.event.pull_request.labels.*.name, 'allow')
```

This should ideally be a temporary solution as it requires manually reviewing and assigning the label for each such pull request.

Security by monitoring requires due diligence in multiple areas. Results of scans need to be reviewed and acted upon. The same is true for failed pull request processing. Where doable, though, as much of the *reviewing* and *responding* should be automated. Leveraging GitHub functionality like Dependabot scans that can generate automated pull requests is a useful step in that direction.

Here’s one final point about remediation/prevention strategies for pull requests. Keep in mind that these strategies will only be effective from the point in time where you enact them. Any older changes that are still outstanding and cannot go through the new strategies will need to be handled separately. That may include closing them and asking users to submit new ones, which can then be validated by the newer strategies.

# Conclusion

Workflows and actions in GitHub provide a convenient means of achieving automatic processing that is highly integrated into your repositories and your execution environments. But that high integration also carries a high risk of allowing security vulnerabilities and a high degree of responsibility for doing due diligence to keep them out.

You can help to reduce the chances of these risks through configuring your repositories and action execution environments to require oversight and prevent others from making modifications unless approved.

You can use good design principles to encapsulate information that should not be exposed in secrets and limit access for users and automated processes through appropriately scoped tokens. And you can plan for ways to prevent common attacks, such as untrusted input from being introduced at runtime.

But all of this preparation can only get you so far and doesn’t guard against vulnerabilities being introduced through dependencies or other actions. For that you need review and regular scanning to identify issues.

You must also guard against malicious or accidental issues being introduced through the GitHub collaborative features, namely, pull requests. GitHub Actions by default is set up to not execute workflows in the target repository’s environment. But it is possible to override that via the `pull_request_target` event trigger, and that can lead to increased exposure.

Finally, you must remember that workflows will be executing on a runner somewhere and that any code—workflows, actions, source code—on that runner has some level of access to gather information from the environment, work with the filesystem, etc. So, it is critically important to monitor the incoming changes and understand what effect they may have on the current code and what they may be trying to do on the runner system.

In the next chapter, we’ll continue to explore how to understand more about what is going on with your workflows. [Chapter 10](ch10.html#ch10) will cover the techniques and functionality available for you to troubleshoot, track, and observe as your workflows run, through the use of monitoring, logging, and debugging.

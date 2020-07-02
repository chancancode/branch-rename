# Git Branch Rename

So, you want to rename a branch on Git/GitHub. The most common case of this is
to rename the default "master" branch to something else, such as "main". We are
going to use this as our example, but you may have a different use case in
mind. If that's the case, just replace "master" and "main" to the source and
target branches you have in mind.

## Table of Contents

* If you are starting a brand new project, see [New Project](#new-project).

* For small personal projects, see [Simple Migration](#simple-migration).

* For work or open-source projects, see [Gradual Migration](#gradual-migration).

* For projects you don't control, see [Local Migration](#local-migration).

## New Project

If you are starting a brand new project, you can follow these steps:

1. [Create a new repository on GitHub](https://github.com/new) to host your
   project with the following settings:

   * **Repository template**: No template
   * **Initialize this repository with a README**: Leave unchecked
   * **Add .gitignore**: None
   * **Add a license**: None

   If you wish to use any of these features (i.e. with settings different than
   the above), that is not a problem. However, GitHub will create the default
   "master" branch for you. In that case, clone the repository locally and
   follow the steps in the [**Simple Migration**](#simple-migration) section
   instead.

2. Initialize an empty Git repository:

   ```shell
   $ mkdir my-git-project
   $ cd my-git-project
   $ git init .
   ```

3. Create the "main" branch:

   ```shell
   $ git checkout -b main
   ```

4. Work on your project and commit some changes. For example, add a README:

   ```shell
   $ echo "# My Git Project" >> README.md
   $ git add README.md
   $ git commit -m "Initial commit"
   ```

5. Push your commits to GitHub:

   ```shell
   $ git remote add origin https://github.com/your-username/my-git-project.git
   $ git push -u origin main
   ```

That's all! Since "main" was the first branch you pushed to GitHub, it will be
set correctly as the default branch. We also never committed any changes to the
"master" branch locally, it effectively never existed so there is nothing left
to clean up. Enjoy your new project!

## Simple Migration

If you have a small personal project and you are comfortable with just doing
the rename right away, here are the steps you can take.

Note that if this is a [GitHub Pages](https://pages.github.com) repository and
the content is stored on the "master" branch, then you cannot perform the
migration at this time since GitHub Pages does not seem to support alternative
default branch names yet. If you want, you may follow the steps in the [**Local
Migration**](#local-migration) or [**Gradual Migration**](#gradual-migration)
section partially to get your project ready.

1. Pull the latest commits from the "master" branch into your local repository:

   ```bash
   $ cd my-git-project
   $ git checkout master
   $ git pull origin master
   ```

2. Rename the local "master" branch to "main":

   ```bash
   $ git branch -m main
   ```

3. Push the "main" branch to GitHub:

   ```bash
   $ git push -u origin main
   ```

4. [Change the default branch on GitHub][change-default-branch] to "main".

5. If there are existing pull requests open against the "master" branch that
   you would like to keep, [update their base branch][change-pr-base-branch] to
   the "main" branch. Any unchanged pull requests will be closed automatically when we
   delete the remote "master" branch on GitHub. If you have multiple pull requests that require a base branch update, consider using the [retarget_prs](https://github.com/ethomson/retarget_prs) utility to automate this change.

6. Delete your local "master" branch:

   ```bash
   $ git branch -d master
   ```

7. Delete the remote "master" branch:

   ```bash
   $ git push origin :master
   ```

At this point, the migration is complete. However, as you work on the project,
you may discover additional settings that you need to tweak to account for the
rename. For example, you may have update the branch names in your CI config.

## Gradual Migration

If you have a work or open-source repository with multiple contributors and
forks, you may prefer to perform the migration gradually.

These steps are for you if you find yourself in a similar scenario, where it is
important to give everyone ample of time to prepare for the migration, changing
local configs, adapting workflows, scripts and other automation to ensure a
seamless migration.

This gradual migration plan is intended to be spread out over a long period of
time – weeks, or months. It uses all available tools on GitHub to provide as
much advance notice as possible.

For most organizations, this plan may be overly cautious and some of the steps
may not be necessary. On the other hand, for a popular open-source project, it
may not be feasible to get to the end and fully deprecate and remove the legacy
"master" branch at all, due to compatibility requirements. Treat this plan as a
template and adapt it for your own needs.

Unlike the other sections, the steps here are a bit less exact and is intended
for someone with a some prior experience with Git and GitHub. In practice, you
will probably run into situations that causes deviations from the plan which
would require some manual repair and adjustments. For example, pushes during a
partial GitHub outage may cause the two branches to diverge and requires you
to determine how to best reconcile the differences.

### Phase 1: Mirror "master" and "main"

The goal of this phase is to make it _possible_ to use the "main" branch name
as an alternative. This allows some early adopters to start testing the new
setup.

1. As always, make sure your local "master" branch is up-to-date.

2. Create the new "main" branch:

   ```bash
   $ git checkout -b main
   ```

3. Add a [GitHub Actions](https://github.com/features/actions) workflow file at
   **.github/workflows/mirror-master-and-main.yml** with the following:

   ```yaml
   name: Mirror "master" and "main" branches
   on:
     push:
       branches:
         - master
         - main

   jobs:
     mirror:
       runs-on: ubuntu-latest
       steps:
         - name: Mirror to "master"
           uses: zofrex/mirror-branch@v1
           with:
             target-branch: master
             force: false
         - name: Mirror to "main"
           uses: zofrex/mirror-branch@v1
           with:
             target-branch: main
             force: false
   ```

4. Commit and your changes to the local "main" branch.

5. Push your changes to the remote "main" branch:

   ```bash
   $ git push -u origin main
   ```

If things are working correctly, you should see the same commit pushed to the
"master" branch shortly. You can monitor the progress and unexpected errors in
the "Actions" tab in your repository.

#### About GitHub Actions

You do not have to be already using GitHub Actions for this to work. You do not
need to activate or enable the feature for your repository, simply pushing the
workflow file to the is sufficient. All open-source repositories have unlimited
GitHub Actions minutes, and all personal and team accounts comes with 2000-3000
free GitHub Actions minutes for private repositories.

If you already regularly uses up your free minutes, this may slightly increase
your GitHub bills, but in practice, the workflow we added runs very quickly so
the impact is very minimal. For reference, GitHub Actions are billed at $0.008
USD per minute for private repositories, after the free quota is exhausted.

This workflow will be triggered when commits are pushed to either the "master"
or "main" branch. It uses the [Mirror Branch action][mirror-branch-action] to
update the branches using GitHub's API.

#### Interaction with Branch Protection

Unfortunately, if you have enabled [branch protection][branch-protection] on
the "master" branch, the push from the mirroring action may be rejected. For
example, if you had enabled "Require pull request reviews before merging", then
this would not work as the changes are expected to be submitted via a pull
request with reviews.

A possible workaround is to disable the "Include administrators" checkbox in
the branch protection settings for the "master" branch and configure the script
to push the commits as an administrator:

1. Login as an administrator on GitHub. You may also want to consider creating
   a new account specifically for this purpose.

2. Generate a [personal access token][personal-access-token] with the "repo"
   scope (and the "public_repo" scope, if needed).

3. [Add it as a secret][add-secret] to the repository.

4. Pass the token to the [Mirror Branch action][mirror-branch-action]:

   ```yaml
   - name: Mirror to "master"
     uses: zofrex/mirror-branch@v1
     with:
       token: ${{ secrets.DEPLOY_TOKEN }}
       target-branch: master
       force: false
   - name: Mirror to "main"
     uses: zofrex/mirror-branch@v1
     with:
       token: ${{ secrets.DEPLOY_TOKEN }}
       target-branch: main
       force: false
   ```

   Here, `DEPLOY_TOKEN` is the name you picked from step 3.

#### Interaction with Other GitHub Action Workflows

By default, when pushing commits from within the GitHub Actions job, it does
not trigger additional GitHub Actions workflow to run. For example, when a
contributor pushes to the "main" branch, it will trigger any GitHub Actions
that normally runs on the "main" branch's push events, including the one we
added here. However, when our mirror workflow pushes the same commit to the
"master" branch, it will not trigger any workflow that normally runs on the
"master" branch's push events.

For this reason, you may want to update existing workflows that runs on the
"master" branch to also run on the "main" branch, as we did in our mirror
workflow file (the `on.push.branches` config key). This is the recommended
approach as it ensures only a single build per push.

Alternatively, if it is important to you that workflows are triggered by pushes
from the mirror workflow, you can accomplish this by supplying a personal
access token to the [Mirror Branch action][mirror-branch-action] by following
the steps in the [previous section](#interaction-with-branch-protection).

With this, the mirror workflow will authenticate with GitHub using the personal
access token when updating commits, triggering any workflows as if a regular
user had pushed those commits. This does not change the author or committer on
the commits.

#### Real World Example

See [this commit][p1-rwe-commit] and [the resulting workflow run][p1-rwe-run]
for an example of this working in action. Note that the code in the commit may
be outdated by the time you read this – refer to the above for the latest
instructions.

#### Next Steps

After verifying that everything is working as intended, you can start inviting
the early adopters to start pushing to the "main" branch. The easiest way to do
this is to rename the local "master" branch:

```bash
$ cd my-git-project
$ git checkout master
$ git branch -m main
$ git branch -u origin/main
```

This would also be a good time to start changing any automation or external
services to the "main" branch to ensure that everything is working as expected.

### Phase 2: Change the default branch to "main", deprecate "master"

After verifying the viability of the rename during the previous phase, the goal
of this phase is to set "main" as the default branch and start issuing
deprecation warnings when the legacy "master" branch is used.

1. [Change the default branch on GitHub][change-default-branch] to "main".

2. Add a [GitHub Actions](https://github.com/features/actions) workflow file at
   **.github/workflows/deprecate-master-branch.yml** with the following:

   ```yaml
   name: Deprecate "master" branch
   on:
     push:
       branches:
         - master
     pull_request:
       branches:
         - master

   jobs:
     on-push:
       runs-on: ubuntu-latest
       if: ${{ github.event_name == 'push' }}
       steps:
         - name: Deprecation
           uses: peter-evans/commit-comment@v1
           with:
             body: |
               Hello @${{ github.event.sender.login }}!

               I see that you have pushed some commits to the "master" branch. We are in the process of renaming the "master" branch to "main" in this repository.

               :warning: **The "master" branch is deprecated and will be removed from this repository in the future.**

               Please migrate your local repository by renaming the "master" branch to "main":

               ```bash
               $ cd my-git-project
               $ git checkout master
               $ git branch -m main
               $ git branch -u origin/main
               ```

               Before merging pull requests, ensure their base branch is set to "main" instead of "master". For more information on how to do this, refer to [this GitHub support article][1].

               [1]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-base-branch-of-a-pull-request

     on-pull-request:
       runs-on: ubuntu-latest
       if: ${{ github.event_name == 'pull_request' }}
       env:
         DEPRECATION_MESSAGE: |
           Hello @${{ github.event.sender.login }}!

           I see that you have opened a pull request against the "master" branch. We are in the process of renaming the "master" branch to "main" in this repository.

           :warning: **The "master" branch is deprecated and will be removed from this repository in the future.**

           Please migrate your local repository by renaming the "master" branch to "main":

           ```bash
           $ cd my-git-project
           $ git checkout master
           $ git branch -m main
           $ git branch -u origin/main
           ```

           Please also set the base branch for this pull request to "main" instead of "master". For more information on how to do this, refer to [this GitHub support article][1].

           [1]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-base-branch-of-a-pull-request
       steps:
         - name: Deprecation
           if: ${{ github.event.pull_request.head.repo.fork == false }}
           uses: peter-evans/create-or-update-comment@v1
           with:
             issue-number: ${{ github.event.number }}
             body: ${{ env.DEPRECATION_MESSAGE }}
         - name: Deprecation
           if: ${{ github.event.pull_request.head.repo.fork == true }}
           run: |
            echo "$DEPRECATION_MESSAGE"
            echo '::error::Please set the base branch for this pull request to "main" instead of "master".'
            exit 1
   ```

3. Commit and push your changes to the "main" branch.

We added a workflow file that triggers whenever a contributor pushes to or
opens a pull request against the "master" branch.

The workflow adds a comment to the commit or pull request, notifying the
contributor that the "master" branch has been deprecated, along with the steps
they need to take to migrate their local repository and changes they need to
make it to the pull request.

It is recommended that you customize the messages with additional information
relevant for your organization. For example, you may want to include a link to
a tracking issue for additional context, or ways for the contributor to ask for
additional assistance if needed.

#### Limitations of GitHub Actions in Forks

Unfortunately, due to limitations of GitHub Actions, it is not possible to add
a pull request comment from the workflow when the pull request originated from
a fork, which is very common in open-source repositories. As a workaround, the
workflow prints the deprecation message to the logs and fails the build.

#### Real World Example

See [this commit][p2-rwe-commit] and [this pull request][p2-rwe-pr] for an
example of this working in action. Note that the code in the commit may be
outdated by the time you read this – refer to the above for the latest
instructions.

#### Next Steps

This would be a good time to start updating any internal and external links to
the "master" branch. A common example would be automatically generated links to
source code from the API documentation.

### Phase 3: Complete the migration

After a successful phase 2 rollout, it is time to plan for completing the
migration. However, what completion means is going to be different depending on
your situation.

For private work repositories, the legacy "master" branch can likely be removed
from the repository after giving team members a few weeks to migrate. Don't
forget to remove the workflow files as well.

For open-source repositories with lots of contributors, you may want to move a
lot slower. Monitor the Actions tab of the repository to see how often the
deprecation workflow is triggered. When the activity diminishes, it may be good
indication that the legacy "master" branch is no longer needed.

#### Soft Removal

As an alternative to removing the branch right away, you may want to consider
pushing a final commit to the branch, removing all files but leave behind a
README file explaining that branch has been moved, along with the steps they
need to take in order to migrate their local repository.

#### URL Considerations

It is also important to consider URL breakages, as links that points to the
files on the "master" branch will stop working once the branch is removed,
including with the soft-removal method described above.

The impact of this has to be evaluated contextually, but it is important to
note that the scope of the breakage is fairly limited, as this only directly
impacts URLs that links to the "master" branch directly. Links pointing to the
active development branch of repository is quite fragile (especially with line
numbers) and often breaks for other reasons.

For example, a refactor that moves around files or switching from JavaScript to
TypeScript would invalidate these URLs. This sort of activity is fairly common
on an active code repository and is usually performed without taking the URL
consideration in mind. For this reason, it is considered a best practice to use
SHA-based [permanent links][permanent-links] in most situations. These links
are unaffected by the branch rename.

#### Packages Considerations

For repositories containing *installable packages*, there are some additional
considerations. Many package managers allow for installing packages from a Git
repository. For example, in npm and yarn, dependencies can be a string like
"username/repo" or "username/repo#branch" in lieu of version range. Likewise,
GitHub Actions are installed using repository and branch references, and it is
a relatively common practice to point an action at the "master" branch.

In these cases, the decision on whether to remove the legacy "master" branch
has to be made carefully. Here are some examples of things to investigate
and consider:

* When the branch is omitted from the specifier (e.g. "username/repo"), does
  the package manager in your ecosystem hard-code the default to "master" on
  the client, or does it respect the remote HEAD ref?

* Does the package manager use a lockfile, and if so, does it serialize the
  branch name (as opposed to the resolved SHA) into the lockfile?

* Does the package manager maintain a cache of cloned repositories, and if so
  do they recover gracefully to a remote branch disappearing?

The answer to these questions affects the potential impact to your end-users
if the legacy "master" branch is removed from the repository. For some, the
end-state of the migration may be to keep the "master" branch permanently as a
read-only mirror, or it may be sufficient to freeze the branch's content and
stop providing updates there. For others, the potential breakage maybe small
enough that it is can be easily justified.

While the workflows added in phase 2 are effective for deprecating _writes_ to
the legacy "master" branch. Unfortunately, Git and GitHub do not offer the
ability to do the same for _reads_ to the branch.

However, you may be able to use features from the package manager to accomplish
a similar result. For example, instead of mirroring the "main" branch to the
"master" branch exactly, you could add a post-install hook to the version on
"master" to issue the deprecation message for any potential consumers.

#### A Concrete Example

Using the Node.js ecosystem as an example, we can put these considerations into
context, based on preliminary testing with npm and yarn classic (v1). If you
found different results in your own testing, or with other package managers
(pnpm, yarn v2, etc), please file an issue here.

* The popular package managers in the ecosystem support installing packages
  from git, by specifying "username/repo" or "username/repo#branch" in the
  dependencies section of package.json.

* When a branch name is not specified, the popular package managers defaults to
  the remote HEAD ref, which is set by the default branch on GitHub, instead of
  hardcoding to the "master" branch in the client.

* The popular package managers uses a lockfile by default. They serialize the
  resolved SHA into the lockfiles, as long as the commits referred to by these
  SHAs remain reachable on the remote repository, they will install just fine
  when using a lockfile. In the case of a branch rename, this is not an issue,
  as the history and commits will remain intact.

* When installing without a lockfile, the popular package managers will fetch
  the latest commit from the repository. Renaming the remote branch did not
  appear to cause any issues, either because the repositories are not cached,
  or the clients are able to recover gracefully from a "missing" remote branch.

* Post-install hooks are supported. However, yarn classic [hides the output
  produced by these scripts](https://github.com/yarnpkg/yarn/issues/5476) if
  they are successful (exit cleanly with exit code 0). However, when they fail,
  they output _is_ shown to the user.

* Generally speaking, the ecosystem has strong norms and expectations around
  adhering to [semantic versioning][semver]. Usually, breaking changes are only
  expected on major version bumps.

* By using a Git dependency instead of specifying a semver range, they are
  explicitly opting out of the normal semver guarantee, and breakages are to be
  expected. No one could reasonably expect that pointing a dependency to the
  active development branch without using a lockfile will result in a stable
  system.

Given these findings, here is a concrete proposal for a maximally graceful
completion plan:

1. Stop providing updates to the "master" branch by removing the mirroring
   workflow added in phase 1.

2. Push a commit the "master" branch, updating the README as well as adding a
   deprecation message to the runtime code (the `main` entry point):

   ```js
   // index.js

   let command = 'npm update my-git-project';

   if (process.env.npm_execpath && process.env.npm_execpath.indexOf('yarn') !== -1) {
     command = 'yarn upgrade my-git-project';
   }

   console.warn(
     `You are running a deprecated copy of the "my-git-project" installed ` +
     `from the "master" branch of our repository. The "master" branch is ` +
     `deprecated and no longer receives any updates. The branch will soon ` +
     `be removed from our repository, at which point the package will fail ` +
     `to install.\n\n` +
     `To fix this issue, please modify your package.json and change the ` +
     `"my-git-project" dependency from "my-repo/my-git-project#master" to ` +
     `"my-repo/my-git-project" or a valid semver range. After making the ` +
     `change, run "${command}" to update the package.`
   );

   // ...rest of index.js
   ```

3. When releasing the next major version of the package, push another commit to
   the "master" branch, remove all files from the branch, leaving behind only a
   minimal README file, package.json and index.js:

   ```js
   // package.json
   {
     "name": "my-git-project",
     "scripts": {
       "postinstall": "node index.js"
     }
   }
   ```

   ```js
   // index.js

   let command = 'npm update my-git-project';

   if (process.env.npm_execpath && process.env.npm_execpath.indexOf('yarn') !== -1) {
     command = 'yarn upgrade my-git-project';
   }

   console.error(
     `You have installed "my-git-project" from the "master" branch of our ` +
     `repository. The "master" branch has been official retired.\n\n` +
     `To fix this issue, please modify your package.json and change the ` +
     `"my-git-project" dependency from "my-repo/my-git-project#master" to ` +
     `"my-repo/my-git-project" or a valid semver range. After making the ` +
     `change, run "${command}" to update the package.`
   );

   process.exit(1);
   ```

4. After some time, the master branch can be removed from the repository. This
   does not constitute a breaking change, as the package already ceased to
   function as of the previous commit. It was just a courteous message to ease
   confusion and provide actionable instructions for fixing the issue.

For most projects and organizations, this amount of notice is probably not
necessary or warranted, but it showcases the available tools and techniques,
and demonstrates that there can be a good migration path even under very strict
compatibility requirements. As always, use these steps as a template and tailor
them to your own needs.

## Local Migration

Finally, if you are working on a repository you don't control, and you would
like to refer to your local branch with a different name without making any
changes to the upstream project, you can rename your local "master" branch to
"main" with these steps:

```shell
$ cd my-git-project
$ git checkout master
$ git branch -m main
$ git branch -u origin/master
```

This renames the local branch to "main" but sets the remote tracking branch to
"master".

## License

This content in this repository, including this documentation and code examples
are licensed under the [CC0 "No Rights Reserved"][cc0-license] public domain
license. Feel free to reproduce and adapt this work into your own proposals,
documentation, etc.

Attribution is not necessary. However, this guide receives constant updates to
reflect current best-practice and solutions based implementation feedback, so
a reference to this repository may be helpful.

[add-deploy-key]: https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys

[add-secret]: https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets-for-a-repository

[branch-protection]: https://help.github.com/en/github/administering-a-repository/configuring-protected-branches

[cc0-license]: https://creativecommons.org/share-your-work/public-domain/cc0/

[change-default-branch]: https://help.github.com/en/github/administering-a-repository/setting-the-default-branch

[change-pr-base-branch]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-base-branch-of-a-pull-request

[checkout-action]: https://github.com/actions/checkout

[generate-ssh-key]: https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

[mirror-branch-action]: https://github.com/marketplace/actions/mirror-branch

[p1-rwe-commit]: https://github.com/chancancode/ember-concurrency-async/commit/afbacf7088e473dd056138109e00d8749b95b2d0

[p1-rwe-run]: https://github.com/chancancode/ember-concurrency-async/runs/788797792?check_suite_focus=true

[p2-rwe-commit]: https://github.com/chancancode/ember-concurrency-async/commit/7f3509b4363f3fcdb3275ec136c4b5766f39c192

[p2-rwe-pr]: https://github.com/chancancode/ember-concurrency-async/pull/5

[permanent-links]: https://help.github.com/en/github/managing-files-in-a-repository/getting-permanent-links-to-files

[personal-access-token]: https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line

[semver]: https://semver.org

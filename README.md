# Git Branch Rename

So, you want to rename a branch on Git/GitHub. The most common case of this is
to rename the default "master" branch to something else, such as "main". We are
going to use this as our example, but you may have a different use case in
mind. If that's the case, just replace "master" and "main" to the source and
target branches you have in mind.

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
   follow the steps in the **Simple Migration** section instead.

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
default branch names yet. If you want, you may follow the steps in the "Local
Migration" or "Gradual Migration" section partially to get your project ready.

1. Pull the latest commits from the "master" branch into your local repository:

   ```bash
   $ cd my-git-project
   $ git checkout master
   $ git pull origin master
   ```

2. Create the new "main" branch, using the "master" branch as a starting point:

   ```bash
   $ git checkout -b main
   ```

3. Push the "main" branch to GitHub:

   ```bash
   $ git push -u origin main
   ```

4. [Change the default branch on GitHub][change-default-branch] to "main".

5. If there are existing pull requests open against the "master" branch that
   you would like to keep, [update their base branch][change-pr-base-branch] to
   the "main" branch. Otherwise, they will be closed automatically when we
   delete the remote "master" branch on GitHub.

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
as an alternative. This allows some early adaopters to start testing the new
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
         - name: Checkout
           uses: actions/checkout@v2
           with:
             fetch-depth: 0
         - name: Push
           run: |
             git push origin HEAD:master
             git push origin HEAD:main
   ```

4. Commit and your changes to the local "main" branch.

5. Push your changes to the remote "main" branch:

   ```bash
   $ git push -u origin main
   ```

If things are working correctly, you should see the same commit pushed to the
"master" branch shortly. You can monitor the progress and unexpected errors in
the "Actions" tab in your repository.

[change-default-branch]: https://help.github.com/en/github/administering-a-repository/setting-the-default-branch

[change-pr-base-branch]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-base-branch-of-a-pull-request

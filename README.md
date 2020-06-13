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

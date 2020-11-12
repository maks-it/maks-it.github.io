# Git Tutorial

Sourcs:
* [Git за полчаса: руководство для начинающих](https://proglib.io/p/git-for-half-an-hour)
* [В чем разница между git add ., add -A, add -u и add *?](https://ru.stackoverflow.com/questions/431839/%D0%92-%D1%87%D0%B5%D0%BC-%D1%80%D0%B0%D0%B7%D0%BD%D0%B8%D1%86%D0%B0-%D0%BC%D0%B5%D0%B6%D0%B4%D1%83-git-add-add-a-add-u-%D0%B8-add)

## Initial configuration

First of all agter git installation we need to set default `username` and `email` globally

```bash
$ git config --global user.name "My Name"
$ git config --global user.email myEmail@example.com
```

## Create new repository

Open visual Studio Code, then open your folder and create workspace, add folder to source control.
Inside terminal, type:

```bash
$ cd "path to your repo"
$ git init
$ git add . # if you want to commit everything. Otherwise use .gitconfig files
$ git commit -m "initial commit" # If you change anything, you can add and commit again...
```

## Add remote to repository

To add a remote, just do

```bash
$ git remote add origin https://...
$ git remote show origin # if everything is ok, you will see your remote
$ git push -u origin master # assuming your are on the master branch.
```

The -u sets an upstream reference and git knows from where to fetch/pull and where to push in the future.

## Status definition

**status** - is one of the most important commands, which shows information about current repository state: whether the information on it is relevant, is there something new that has changed, and so on.

Running git status on our newly created repository should produce:

```bash
$ git status
On branch master
Initial commit
Untracked files:
(use "git add ..." to include in what will be committed)
hello.txt
```

The message indicates that the hello.txt file is untracked. This means that the file is new and the system does not yet know whether it is necessary to monitor the changes in the file or it can simply be ignored. In order to start tracking a new file, you need to declare it in a special way.

## Preparing files

Git has the concept of a staged file scope. You can think of it as a canvas on which to apply the changes that are needed in the commit. At first it is empty, but then we add files (or parts of files, or even single lines) to it with the add command and, finally, we commit everything we need to the repository (create a snapshot of the state we need) with the commit command.

In our case, we only have one file, so let's add it:
```bash
$ git add hello.txt
```

If we need to add everything in the directory, we can use:
```bash
$ git add -A
```

Let's check the status again, this time we should get a different answer:
```bash
$ git status
On branch master
Initial commit
Changes to be committed:
(use "git rm --cached ..." to unstage)
new file: hello.txt
```

The file is ready to commit. The status message also tells us what changes have been made to the file in the staging area - in this case, it's a new file, but the files can be modified or deleted.

## Commit (committing changes)

A commit represents the state of the repository at a specific point in time. It's like a snapshot that we can go back to and see the state of objects at a particular point in time.
To commit the changes, we need at least one staging change (we just created it with git add), after which we can commit:

```bash
$ git commit -m "Initial commit."
```

This command will create a new commit with all staging changes (adding hello.txt file). The -m switch and the "Initial commit." Is a user-generated description of all the changes included in the commit. It is considered good practice to commit frequently and always write meaningful comments.

## Remote repositories

Our commit is now local - it only exists in the .git directory on our filesystem. While the local repository itself is useful, in most cases we want to share our work or deliver code to a server where it will run.

### Connecting to a remote repository

To upload something to a remote repository, you first need to connect to it. In our tutorial, we'll use https://github.com/tutorialzine/awesome-project, but we'd recommend trying to create your own repository on GitHub, BitBucket, or any other service. Registration and installation can take time, but all of these services provide good documentation.
To link our local repository to the GitHub repository, run the following command in a terminal. Please note that you must change the repository URI to your own.

```bash
# This is only an example. Replace the URI with your own repository address.
$ git remote add origin https://github.com/tutorialzine/awesome-project.git
```

A project can have multiple remote repositories at the same time. To distinguish between them, we will give them different names. Usually the main repository is called origin.

### Submitting changes to the server

Now is the time to push our local commit to the server. This process happens every time we want to update data in the remote repository.
The command for this is push. It takes two parameters: the name of the remote repository (we named our origin) and the branch to be modified (master is the default branch for all repositories).

```bash
$ git push origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 212 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/tutorialzine/awesome-project.git
* [new branch] master -> master
```

Depending on the service you are using, you may need to authenticate for the changes to be sent. If everything is done correctly, then when you look at the remote repository using a browser, you will see the file hello.txt

### Cloning a repository

Other GitHub users can now view your repository. They can download data from it and get a fully functional copy of your project using the clone command.

```bash
$ git clone https://github.com/tutorialzine/awesome-project.git
```

A new local repository is created automatically from GitHub as a remote repository.

### Requesting changes from the server

If you've made changes to your remote repository, other users can download the changes using the pull command.

```bash
$ git pull origin master
From https://github.com/tutorialzine/awesome-project
* branch master -> FETCH_HEAD
Already up-to-date.
```

Since there have been no new commits since we cloned the project to ourselves, there are no changes available for download.

## Branching

When developing new functionality, it is considered good practice to work with a copy of the original project, which is called a branch. Branches have their own history and changes isolated from each other until you decide to merge the changes together. This happens for a variety of reasons:
* The already working, stable version of the code is saved.
* Various new functions can be developed in parallel by different programmers.
* Developers can work with their own branches without the risk that the codebase will change due to someone else's changes.
* In case of doubt, different implementations of the same idea can be developed in different branches and then compared.

### Creating a new branch

The master branch in each repository is called master. To create another branch, use the branch <name> command

```bash
$ git branch amazing_new_feature
```

This will create a new branch, for now an exact copy of the master branch.

### Switching between branches

Now if we run branch, we will see two options available:

```bash
$ git branch
amazing_new_feature
* master
```

master is the active branch, it is marked with an asterisk. But we want to work with our “awesome new feature”, so we need to switch to a different branch. To do this, we'll use the checkout command, it takes one parameter - the name of the branch to switch to.

```bash
$ git checkout amazing_new_feature
```

### Merging branches

Our “awesome new feature” will be another text file called feature.txt. We'll create it, add it and commit it:

```bash
$ git add feature.txt
$ git commit -m "New feature complete.”
```

The changes are complete, now we can switch back to the master branch.

```bash
$ git checkout master
```

Now, if we open our project in the file manager, we will not see the feature.txt file, because we switched back to the master branch, where such a file does not exist. To make it appear, you need to use merge to merge branches (apply changes from the amazing_new_feature branch to the main version of the project).

```bash
$ git merge amazing_new_feature
```

The master branch is now up to date. The amazing_new_feature branch is no longer needed and can be deleted.

```bash
$ git branch -d awesome_new_feature
```

## Additionally

In the last part of this guide, we'll cover some additional tricks that can help you.

### Tracking changes made in commits.

Each commit has its own unique identifier in the form of a string of numbers and letters. To see a list of all commits and their IDs, you can use the log command:

```bash
$ git log
commit ba25c0ff30e1b2f0259157b42b9f8f5d174d80d7
Author: Tutorialzine
Date: Mon May 30 17:15:28 2016 +0300
New feature complete
commit b10cc1238e355c02a044ef9f9860811ff605c9b4
Author: Tutorialzine
Date: Mon May 30 16:30:04 2016 +0300
Added content to hello.txt
commit 09bd8cc171d7084e78e4d118a2346b7487dca059
Author: Tutorialzine
Date: Sat May 28 17:52:14 2016 +0300
Initial commit
```

As you can see, the identifiers are quite long, but you don't have to copy them entirely to work with them - the first few characters will be enough. To see what's new in the commit, we can use the show [commit] command

```bash
$ git show b10cc123
commit b10cc1238e355c02a044ef9f9860811ff605c9b4
Author: Tutorialzine
Date: Mon May 30 16:30:04 2016 +0300
Added content to hello.txt
diff --git a/hello.txt b/hello.txt
index e69de29..b546a21 100644
--- a/hello.txt
+++ b/hello.txt
@@ -0,0 +1 @@
+Nice weather today, isn't it?
```

To see the difference between two commits, use the diff command (indicating the gap between commits):

```bash
$ git diff 09bd8cc..ba25c0ff
diff --git a/feature.txt b/feature.txt
new file mode 100644
index 0000000..e69de29
diff --git a/hello.txt b/hello.txt
index e69de29..b546a21 100644
--- a/hello.txt
+++ b/hello.txt
@@ -0,0 +1 @@
+Nice weather today, isn't it?
```

We've compared the first commit with the last one to see all the changes ever made. It is usually easier to use git difftool, as this command launches a graphical client that visually compares all changes.

### Reverting a file to a previous state

Git allows you to revert the selected file to the state at the time of a certain commit. This is done by the familiar checkout command, which we previously used to switch between branches. But it can also be used to switch between commits (this is a fairly common situation for Git - using one command for various, at first glance, loosely related tasks).
In the following example, we will take the hello.txt file and roll back any changes made to it to the first commit. To do this, we will substitute the required commit identifier in the command, as well as the path to the file:

```bash
$ git checkout 09bd8cc1 hello.txt
```

### Fixing a commit

If you misspelled a comment or forgot to add a file and notice it right after committing your changes, you can easily fix it with `commit -amend`. This command will add everything from the last commit to the staging area and try to make a new commit. This gives you the opportunity to correct the comment or add missing files to the staging area.
For more complex fixes, for example, not in the last commit or if you managed to send changes to the server, you need to use `revert`. This command will create a commit that undo the changes made in the commit with the given ID.
The most recent commit can be accessed by the HEAD alias:

```bash
$ git revert HEAD
```

For the rest, we will use identifiers:

```bash
$ git revert b10cc123
```

When undoing old commits, you need to be prepared for conflicts. This happens if the file was changed by another, newer commit. And now git cannot find the lines whose state needs to be rolled back, since they no longer exist.

### Resolving merge conflicts

In addition to the scenario described in the previous paragraph, conflicts regularly arise when merging branches or when submitting someone else's code. Sometimes conflicts are fixed automatically, but usually you have to deal with it manually - decide which code remains and which one needs to be removed.
Let's look at examples where we try to merge two branches called john_branch and tim_branch. Both Tim and John edit the same file: a function that displays the elements of an array.
John uses a loop:

```js
// Use a for loop to console.log contents.
for(var i=0; i<arr.length; i++) {
console.log(arr[i]);
}
```

Tim prefers forEach:

```js
// Use forEach to console.log contents.
arr.forEach(function(item) {
console.log(item);
});
```

They both commit their code to the appropriate branch. Now, if they try to merge two branches, they will get an error:

```bash
$ git merge tim_branch
Auto-merging print_array.js
CONFLICT (content): Merge conflict in print_array.js
Automatic merge failed; fix conflicts and then commit the result.
```

The system was not able to resolve the conflict automatically, which means that the developers will have to do it. The app has flagged the lines containing the conflict:

```bash
<<<<<<< HEAD // Use a for loop to console.log contents. for(var i=0; i<arr.length; i++) { console.log(arr[i]); } ======= // Use forEach to console.log contents. arr.forEach(function(item) { console.log(item); }); >>>>>>> Tim's commit.
```

Above the ======= separator we see the last (HEAD) commit, and below it - the conflicting one. This way we can see how they differ and decide which version is better. Or even write a new one. In this situation, we will do just that, rewrite everything, remove the separators, and let git know that we are done.

```js
// Not using for loop or forEach.
// Use Array.toString() to console.log contents.
console.log(arr.toString());
```

When everything is ready, you need to commit the changes to complete the process:

```bash
$ git add -A
$ git commit -m "Array printing conflict resolved."
```

As you can see, the process is quite tedious and can be very difficult on large projects. Many developers prefer to use GUI clients to resolve conflicts. (To run you need to type git mergetool).

### Setting up .gitignore

Most projects have files or entire directories that we don’t want (and most likely don’t want to) commit to. We can make sure they don't accidentally end up in git add -A with the .gitignore file

* Create manually a file called .gitignore and save it in your project directory.
* Within the file, list the names of the files / folders to be ignored, each on a new line.
* The .gitignore file must be added, committed, and uploaded to the server like any other file in the project.

Here are some good examples of files to ignore:
* Logs
* Build system artifacts
* node_modules folders in node.js projects
* Folders created by IDEs like Netbeans or IntelliJ
* Various developer notes.

A .gitignore file excluding all of the above would look like this:

```bash
*.log
build/
node_modules/
.idea/
my_notes.txt
```

The slash at the end of some lines means a directory (and the fact that we recursively ignore all of its contents). The asterisk, as usual, means pattern.

## Usefull commands

Prune remote branches during fetch:
```bash
$ git fetch origin --prune
```

Rebase local branch when pulling:
```bash
git pull origin master --rebase
```

Enable push --force:
```
$ git push origin master --force
```

Checkout and create new branch if not exists:
```bash
$ git checkout -b <branch>
```

Fetch and merge from remote origin with local repository:
```bash
git pull origin master --allow-unrelated-histories
```

Reset local repository to remote:

>**Important**: If you have any local changes, they will be lost. With or without `--hard` option, any local commits that haven't been pushed will be lost.[*]
If you have any files that are not tracked by Git (e.g. uploaded user content), these files will not be affected.

```bash
$ git fetch --all
```

```bash
$ git reset --hard origin master
$ git pull origin master --rebase
```

Safe way to delete commitments history:
>Deleting the .git folder may cause problems in your git repository. If you want to delete all your commit history but keep the code in its current state, it is very safe to do it as in the following:
```bash
$ git checkout --orphan latest_branch
$ git add -A # Add all the files
$ git commit -am "commit message"
$ git branch -D master # Delete the branch
$ git branch -m master # Rename the current branch to master
$ git push origin master --force # Finally, force update your repository
```

>PS: this will not keep your old commit history around

Change git default editor:
>You can set it from the command line or in your .gitconfig
``` bash
    git config --global core.editor vim
```

Change commit message:
>If the commit only exists in your local repository and has not been pushed to GitHub, you can amend the commit message with the command.
```bash
git commit --amend
```

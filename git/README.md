# Detailed Usage of Important Git Commands
## Initialization
#### Create first remote repository and push local files into it
Follow the following steps to create a repository at the first time:

0. Create a new empty repository online in your browser;
1. Go to your local folder which you want to push to remote repo. Then initialize it by
```shell
git init .
```
Note for the dot in the end. If you create your repo with initial files such as readme or gitignore file, you need to pull the remote changes to local place by
```shell
git pull <your_repo_link>
```
2. Add local files
```shell
git add --all
```
The last option can be:
- *--all*: to add all changes including addition, modification and deletion. That is, if you removed some files locally, this will also remove them remotely later;
- *.* (just one dot): add all newly added files and do NOT include deletion. That is, this will NOT delete any remote files which are deleted locally.
- *filename*: just add one file, e.g., `git add README.md`
Other options are not common.

In the first time, you can use *--all* or *dot*. 

3. Commit your changes
```shell
git commit -m '<first commit>'
```
4. Create the origin master branch in remote repository
```shell
git remote add origin <your_repo_link>
```
Sometimes you may meet this kind of problem:
```shell
fatal: 'origin' does not appear to be a git repository
fatal: The remote end hung up unexpectedly
```
Then you can use 
```shell
git remote set-url origin <your_repo_link>
```


5. Finally push all files to master
```shell
git push -u origin master
```
Here the `-u` option automatically sets that upstream for you, linking your repo to a central one. That way, in the future, Git "knows" where you want to push to and where you want to pull from, so you can use `git pull` or `git push` without arguments to handle codes from master. However, in a large project with more than 1 person involved, it is better to use the full command `git push origin <branch>` to ensure you won't ruin the master. 

#### Remove/Reinitialize local repository
If you want to remove/reinitialize your local repo, such as you find some mistakes in your repo and want to create a new repo and push local files again, you can use
```shell
rm -rf .git
```
and then follow steps to create a new remote repo and push files into it.

#### Clone a repository
Cloning a remote repository entirely to your local place by
```shell
git clone <repo_link>
```

## Branches

#### Create a branch and switch to it
```shell
git checkout -b <branch>
```
The suggested git workflow maintains a "master" branch which is always clean (meaning no development takes place in this branch). So it's better to create a new branch other than master and work in it.
#### Switch to an existing branch
```shell
git checkout <branch>
```
#### Copy one file from another branch to the current branch
```shell
git checkout <branch> <filename>
```
#### Push local branch to remote repository
```shell
git push origin <branch>
```
#### List changes in the branch
To list all local branches
```shell
git branch
```
To list status summary with changed files:
```shell
git status
```
To see all changed but **un-added/un-committed** changes (with all detailed content)
```shell
git diff
```
To see all **added/committed** changes
```shell
git diff --cached
```
To see un-added changes between current branch and another branch (such as master branch):
```shell
git diff master
```
To see only filenames of changes between branches instead of all details:
```shell
git diff --name-only master
```
To list all history commits starting from the latest one:
```shell
git log <--graph>
```
Here `--graph` is optional and used to show commit graph.

#### Delete a branch
To delete a local branch
```shell
git branch -D <branch>
```
To delete a remote branch
```shell
git push --delete origin <remote_branch>
```
or a simpler way with a colon before branch name
```shell
git push origin :<remote_branch>
```
When deleting a remote branch, you may get some error like:
```shell
 ! [remote rejected] new (refusing to delete the current branch: refs/heads/new)
error: failed to push some refs to 'https://github.com/your_branch.git'
```
This means you are now using the branch you want to delete as the default branch to view in github. Now just go to your repo online, and go to settings->branches and change the default branch to view as some other branch such as master, and then do the deletion again. It will work this time.

**NOTE: make sure you do NOT delete your remote master branch!**

## Synchronize

### Git fetch, merge, rebase, pull, pull --rebase, cherry-pick
These commands are all related to apply changes from one branch to another.
- `git fetch`: only download changes from remote branch (doesn't apply changes into the local branch)
- `git merge`: apply changes (downloaded by `fetch` before) into local branch, and merge all commits in the two branches together.
- `git rebase`: similar to `git merge` that also apply changes into your local branch. Difference is that `git base` applies commits in local branch **based on** another branch instead of merging all commits from the two branches.
- `git pull`: download and merge. Simply equal to `git fetch` + `git merge`.
- `git pull --rebase`: download and rebase. Simply equal to `git fetch` + `git rebase`.
- `git cherry-pick <commit-id>`: apply one commit (usually from another branch) to current branch and only merge this commit without any other commits.

### Git merge Vs. rebase Vs. cherry-pick

#### Ref
- [Git pull vs Git rebase](https://stackoverflow.com/questions/36148602/git-pull-vs-git-rebase/36148752)
- [简单对比git pull和git pull --rebase的使用](https://www.jianshu.com/p/478d912946df)

#### Similarity 
All the commands will apply changes. 

#### Difference
- `merge` and `rebase` will get all commits from a branch, while `cherry-pick` only gets a commit (or several commits depending on the input parameters). 
- `merge` simply merges all commits from two branches together; `rebase` applies all commits in current branch based on all the commits downloaded from another branch; `cherry-pick` only applies one commit from another branch into current branch without other commits;

To distinguish between them, we can use this example. Suppose we have this kind of commit graph (you can use `git log --graph` to check commit graph in the branch):
```shell
--A--B--C--D (remote)
     \--E--F (local branch)
```
That is, you start your local work from commit B in remote master, and commit some local changes E and F (not pushed to remote yet). Meanwhile, someone else adds and pushes commits C and D to remote master (so you are two commits ahead of master and two commits behind master). Then, `git merge` will create new commit by "merging" or combining the commits from local and remote branches together like this:
```shell
--A--B--C--D-\ (remote)
     \--E--F--G (local branch)
```
Note that a new commit G is created with combination of previous commits from two branches. Clearly, G contains NO file changes (an empty commit always looks weird), but makes the graph non-linear and confusing.

`git rebase` on the other hand, is different that it holds the graph linear like this:
```shell
--A--B--C--D(remote)--E'--F' (local branch)
```
This means, `git rebase` will merge the remote changes locally and make your local changes **based on** the remote ones, instead of merging local commits with remote ones in `git merge`. After rebase, it will delete your local commits (E and F) and make two new commits (E' and F') with exactly the same changes in them. After pushing local commits remotely, both branches will be linear and clean. 

`git cherry-pick` is totally different and it applies changes from another commit to the local branch. Suppose you are in local branch with latest commit F and then run `git cherry-pick C(commit-id)`, then you will get graph tree like this:
```shell
--A--B--C--D (remote)
     \--E--F--C' (local branch)
```
That is, it applies C into local branch and creates a new commit C' which is exactly the same as C. Note that it doesn't influence other remote commits.

Of course, during applying changes, all these commands may bring conflicts, e.g., one file is changed by more than two people. Then you need to fix the conflicts first.

#### Example
If you have no changes in your local branch and only want to get latest code from remote master, use
```shell
git pull origin master
```
If you made changes and want to see your changes based on latest remote master, use 
```shell
git pull --rebase origin master
```
If you only want to apply some commit on the local branch
```shell
git cherry-pick <commit-id>
```
In some cases you may get conflicts between your local files and corresponding same remote files. It usually because you and someone else changed the same file. To abandon the rebase, simply use
```shell
git rebase --abort
```
and the local branch will be recovered to the pre-rebase status. Similarity, there is also `git merge --abort`. However, there is no abort option for `cherry-pick`. Since it is only one commit, so if you want to abort cherry-pick, simply use `git reset --hard HEAD~` to go back to the previous commit before cherry-pick. 

However, if there were uncommitted worktree changes present when the merge started, `git merge --abort` will in some cases be unable to reconstruct these changes. It is therefore recommended to **always commit or stash your changes before running `git merge` or `git rebase`**.

### Git checkout

#### Ref
- [What is detached HEAD state and how do I get out of it?](https://howtogit.net/recipes/getting-out-of-detached-head-state.html)

To go back to some previous commit (without deleting commits in history)
```shell
git checkout <commit-id>
```
This won't delete any history commits but simply set HEAD as the previous commit.

An intesting thing is that after checkout, it will show that you are in `detached HEAD` state. This means the current HEAD is pointing to the commit you specified, but now you are actually not in the local branch now. Instead, you are in some kind of "virtual" branch. That is, `checkout` won't affect your branches in any way and you are just viewing your history at a certain point in time. So, by `git checkout <branch>` you can easily return to the original state in your branch. 

Therefore, `checkout` is used to check some history commit but don't make changes, or you want to work from that commit liket this:
```shell
git checkout <some-commit-id> # return to previous commit 
git checkout <new-branch> # start a new branch from the commit 
```

### Git reset

#### Ref
- [How to revert a Git repository to a previous commit](https://stackoverflow.com/questions/4114095/how-to-revert-a-git-repository-to-a-previous-commit)

#### git reset --hard
The command `git reset` is used to to remove some previous commits. Depending on the option, it will keep the (file) changes in previous commits (`--soft`) or delete the changes (`--hard`). Note that this is dangerous, so it's better to stash your changes before this so that you can still recover them if you regret.
 
To reset the local branch as some previous commit and abandon all changes after it:
```shell
git reset --hard <commit-id/HEAD/HEAD~2>
```
This is the case that you want to abandon anything from the commit you started. Here `HEAD` is the latest commit, and `HEAD~1` is 1 commit before `HEAD`, `HEAD~2` is the commit that is 2 commits before HEAD. HEAD is `HEAD~0` by default, while `HEAD~` is equal to `HEAD~1` by default. 

To reset the local branch exactly the same as remote master
```shell
git fetch origin
git reset --hard origin/master
```
This is usually used when you want to remove everything from your local branch and start again from current origin master. Actually this is not common, since you can simply create a new branch and pull latest remote master instead of using reset.

Some notes of usage:
- `reset --hard` will **throw away** all commits as well as file changes between the previous commit and current status, and make the previous commit as new HEAD. That is, it will **rewind** your HEAD to the specified commit. So, it is dangerous and don't use it unless you know what you are doing. 
- You can use `git reset --soft` if you only want to go back to some previous and keep changes. 
- Even though some commits and changes are removed by `reset --hard`, they are still there in the git database. Therefore, if you still know the commit ID of your original HEAD, you can still recover to your original branch by `git reset --hard <original-HEAD-commid-id>`.

#### git reset --soft
If you only want to delete some previous commits but still want to keep changes
```shell
git reset --soft <commit-id/HEAD/HEAD~2>
```
This will only remove the commits after the commit you specified. The file changes are still stashed but not committed, and the changed files are in the index (you can use `git status` to check it). This means, your code is still exactly the same as before you reset. If you make new corresponding commits for the changes, you will still get the same branch as before. Therefore, `reset --soft` is very useful if you made too many commits with only very few changes.


### Merge local changes into remote master
An example here:
```shell
git checkout -b <branch_name> # skip this if you are already in a branch (not master)
git add --all
git commit -m 'sth.'
git push (-f) origin <same_branch_name>
git checkout master # return to master
git merge <same_branch_name> # merge branch into master
git add --all # may skip these two lines
git commit -m 'xxxxx' 
git push origin master # push new codes to remote
git branch -d <same_branch_name> # delete the local branch if you want 
git push origin --delete <same_branch_name> # delete remote branch already merged
```

## Gitignore

The file `.gitignore` in the repo is used to control files to be omitted in the index and remote repo.

To untrack a single file that has already been added/initialized to your repository, i.e., stop tracking the file but not delete it from your system use: 
```shell
git rm --cached filename
```
NOTE:
To undo this step, you can use 
```shell
git add filename
```
To untrack every file currently in your .gitignore
```shell
git rm -r --cached .
```
Note for the dot in the end. This removes any changed file from the index (staging area). This command is usually used when you just modify the gitignore file by adding or deleting a format, and want to apply it on all files in the repo. After untracking all files, you can add them again into index by `git add --all`, make a new commit and push them.

## Submodules

#### Ref
- [Git Submodules: Adding, Using, Removing, Updating](https://chrisjean.com/git-submodules-adding-using-removing-and-updating/)
- [Easy way to pull latest of all git submodules](https://stackoverflow.com/questions/1030169/easy-way-to-pull-latest-of-all-git-submodules)
- [How do I remove a submodule](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule/36593218#36593218)

### Add submodule into a repo:
```shell
git submodule add <submodule_link> <path_to_put_submodule_in_your_repo>
```
If you are using new git versions, this command will download all files from the remote submodule to your local place. 

### Update submodule as latest
If you already has local copy of submodule, and want to update your local submodule as the latest remote one (for instance, the remote submodule is newer than yours), you can use
```shell
git submodule update --recursive
```
Note that this only ensure your local submodule is the latest as remote submodule, not the entire remote repo. In your master repo, you still need to commit this change and push it to remote repo.

If that's the **first time** you checkout a repo you need to add `--init`:
```shell
git submodule update --init --recursive
```

### Remove submodules
#### Remove the submodule entry from .git/config 
```shell
git submodule deinit -f path/to/submodule 
```

#### Remove the submodule directory from the superproject's .git/modules directory 
```shell
rm -rf .git/modules/path/to/submodule 
```

#### Remove the entry in .gitmodules and remove the submodule directory located at path/to/submodule 
```shell
git rm -f path/to/submodule
```
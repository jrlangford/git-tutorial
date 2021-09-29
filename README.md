# Git tutorial

This tutorial is focused on a basic rebase and merge strategy whose main goals are the following:

- Force developers to be aware of any potential conflicts introduced in the branch they will merge into.
- Keep an easy to read history of each set of commits that is merged into a branch.

## Requirements

### Packages

- git

### Configuration

Add the following alias to your gitconfig.

```
[alias]
  ...
  ...
  gr = log --graph --pretty=format:'%C(auto) %h - %d %s %C(magenta)(%cr) %C(bold blue)<%an>'
```


## Rebasing


The word rebase means 'to establish a new base for something'. 

In git, we use the `rebase` command to replace the base of a branch. The base of a branch is the commit that was used as a starting point when the branch was first created.

The need to rebase a branch commonly arises when we want to update a branch whose base resides in a past commit of a parent branch so its new base is located at the latest commit of the parent branch.

In order to perform a rebase, the developer must switch to the branch to be rebased and execute the rebase command, passing as argument the new commit to be used as a base. Since in git the name of any given branch is actually a pointer to its latest commit we can directly pass it as an argument to the rebase function.

Example:
```
git checkout feature_branch
git rebase develop
```

This will first switch the current branch to 'feature_branch' and then attempt to change its base commit to the latest commit of 'develop' 

Whenever git detects conflicts during rebasing, it will prompt the developer to fix them.

### Exercises

#### Basic rebasing

Inspect the formatted log of the 'develop' branch.

```
git gr develop
```

You will see the latest commit has the shortened hash "05a3c6b". The one before that one has the shortened hash "f0e656e"


Switch to the 'feature-c' branch of this repo.

```
git checkout feature-c
```

Inspect the formatted log of this branch.

```
git gr
```

You will see the latest commit has the shortened hash "63257f3". The one before that one has the shortened hash "2e791c6".
In a real-world scenario one explanation for this could be that the 'feature-c' branch was created before 'f0e656e' and '05a3c6b' were added to the deveop branch.


Rebase the 'feature-c' branch onto commit 'f0e656e'

```
git rebase f0e656e
```

Inspect the formatted log of this branch.

```
git gr
```

You will see the latest commit, with the description 'Add c.txt', now has a new hash and that its previous commit is 'f0e656e'.

Usually we don't rebase into a commit which is not the latest commit of another branch. Rebase the current branch into `develop`, which points to its latst commit: '05a3c6b'.

```
git rebase develop
```

You will see the latest commit, with the description 'Add c.txt', now has a new hash and that its previous commit is '05a3c6b'.

As you can see, each time we rebase a branch over another branch the hash of the rebased commits changes and the hashes of the parent branch remain unchanged.


## Merging after a successful rebase

After a successful rebase of a given branch, the latest commit of the parent branch will be present as a past commit of the rebased branch.

If you `merge` this branch into its parent in this state with no additional arguments git will perform a 'fast-forward'. This is a strategy where git changes the pointer of the parent branch so it now points to the latest commit of the rebased branch. Think about the result; in this scenario we have no way of identifying which commits were introduced into the parent branch as part of the merge process.

If you instead `merge` this branch into its parent with the `--no-ff` flag git will not perform a 'fast-forward' and force the creation of a 'merge commit'. With this process we can clearly identify sets of commits that are meant ot be introduced into the repository as an atomic group.


### Exercises

#### Merge with fast-forward

Perform a regular merge of the rebased 'feature-c' branch of the previous exercise into 'develop'

```
git checkout develop
git merge feature-c
```

Inspect the formatted log of this branch.

```
git gr
```

As you can see, the new commit from 'feature-c' is now part of the develop branch. Internally, git only updated the `develop` pointer so it now points to the latest commit of 'feature-c'. We can verify this by looking at the pointers in the log: `91649f0 -  (HEAD -> develop, feature-c) Add c.txt ...`


#### Merge without fast-forward

Reset the develop branch to the state of your local copy of the remote develop branch.

```
git checkout develop
git reset --hard origin/develop
```

Inspect the formatted log to verify its state matches its own state before the merge of the previous exercise.

```
git gr
```

Merge the 'feature-c' branch with `merge --no-ff`.

```
git merge --no-ff feature-c
```

Git will create a 'merge commit' and prompt you to customize the merge message. Leave it as it appears.

Inspect the formatted log of this branch.

```
git gr
```

Notice how the printed graph explicitly shows the point where the set of features introduced by the rebased 'feature-c' branch diverges from the develop branch and the point where the set later converges back into the develop branch through a merge commit.


## Troubleshooting

### A merge is performed when `git pull` is executed

`git pull` executes `git fetch` and `git merge` in a single command.

You can avoid this problem by running the commands separately and then manually resolving any conflicts.

Example:

```
git checkout develop
git fetch origin develop
git rebase -r develop origin/develop
```

After this steps are executed, your branch will contain the latest changes of the remote and your local changes will have been applied on top of them.

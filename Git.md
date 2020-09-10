# Git

[toc]

## How to install git

```bash
brew install git
git --version
```

## How to generate SSH key for GitHub authorization

See Git.md

## How to access and copy public SSH key

See Git.md

## How to upload your public SSH key to Github

1. Login to your GitHub account and go to https://github.com/settings/profile
2. Click on “SSH and GPG Keys” to load the SSH key management page.
3. Click on “New SSH key”
4. Enter an appropriate title name.
5. Paste the public SSH key in the key text box.
6. Click on “Add SSH key”

## Test your GitHub authorization

```bash
git clone git@github.com:<Name>/<Name>.git
# or
ssh -T git@github.com
# output will be
# Hi your_user_name! You've successfully authenticated, but GitHub does not provide shell access.
```

## Add your SSH key to ssh-agent

See Git.md

## Misc

```bash
cd <folder>
git init
touch .gitignore
touch .gitattributes
touch README.md
git status
git add
git commit -m "message"

git config --global user.name "your name"
git config --global user.name
git config --global core.autocrlf input

git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit # git ci in command line
git config --global alias.st status

// setup ssh (below section)

git push origin master // first time
git push
git pull
git clone
```

## Flow

http://marklodato.github.io/visual-git-guide/index-en.html

- WD -> S -> H (Commit snapshot also called HEAD or History)
- Reset overwrites S with the last Commit on HEAD. You undo a Add with a Reset.
- Checkout overwrites WD with a copy in S. You undo a edit in WD with checkout.

<img src="/Users/mdeliw/Documents/Info/image-20191027201605774.png" alt="image-20191027201605774" style="zoom:33%;" />

## Remotes

```bash
# create remote from local
brew install hub
cd localrepo
git init
git add .
git commit -m "first commit"
hub create
git push -u origin HEAD
git push --set-upstream origin master

git clone <url>
git remote # show remote handle
git remote -v # shows remote url for fetch and push
git remote show <remote> # see more information

# add remote
git remote add SHORTNAME URL
git remote add pb https://github.com/paulboone/ticgit
git remote -v
git fetch pb

# fetch and pull from remote
git fetch <remote> # note that fetch brings to staging, not into working directory
git pull # fetches and merges into working directory

# push to remote
git push <remote> <branch>
git push origin master

# rename remote
git remote rename <oldname> <newname>
git remote rename pb paul

# remove remote
git remote remove <remote>
```

## Tagging

```bash
git tag # list
git tag -l "v1.8.5*" # shows only tags with v1.8.5.*

# create tag
git tag -a v1.4 -m "tag comment"
git tag

# lightweigh tag, only stores checksum
git tag v1.4-lw
git tag
git show v1.4-lw

# tag a previous commit
git tag -a v1.4 <some-prefix-of-commit-checksum>
git tag -a v1.4 9fceb02

# push tag
git push origin v1.4 # a generic git push will not push tag

# delete tag
git tag -d v1.4-lw # delete from local
git push origin --delete v1.4 # delete from remote

# checkout tag
git checkout v1.4 # just for exploration and discarding
git checkout -b version2 v1.4
```

## Branches

https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell

```bash
git init # creates the master branch in the origin remote
git branch newbranchname # create
git checkout newbranchname # switch to it
git checkout -b newbranchname # create and switch

# simple flow
git checkout master
git checkout -b devbranch
vim index.html
git commit -a -m "create new footer"
git checkout master
git checkout -b hotfix
vim index.html
git commit -a -m "fix email address"
git checkout master
git merge hotfix
git branch -d hotfix
git checkout devbranch
vim index.html
git commit -a -m "finish the new footer"
git checkout master
git merge devbranch # conflict
git add index.html
git commit -m "Final commit"
git branch -d devbranch

# management
git branch # list of current branches
git branch -v # last commit on each branch
git branch --merged # list of branches merged into the branch you're on.
git branch --no-merged # list of branches that contain work you haven't yet merged in. 
git branch --no-merged master # what is not merged into the master

# remote branches
git ls-remote <remote> # list of remote branches
git remote show <remote>

# remote branches are notated as <remote>/<branch>

# synchronize local from remote
git fetch origin

# push branch to remote
git push origin mybranch
# someone else
git fetch origin # will get origin/mybranch on local
git checkout -b copyofmybranch origin/mybranch # now has a local branch o work on that starts from origin/mybranch

# delete remote branch
git push origin --delete mybranch

# rebasing - integrate changes from one branch into another
# use merge or rebase which provides cleaner history
```

## Rebase

To integrate changes from one branch into another, you can use `merge` or `rebase` which offers a cleaner history. You have the following:

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307225817889.png" alt="image-20200307225817889" style="zoom:25%;" />

A merge would do three-way merge and create C5.

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307225905481.png" alt="image-20200307225905481" style="zoom:25%;" />



A rebase would do the following:

```bash
git checkout experiment
git rebase master
```

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307225949129.png" alt="image-20200307225949129" style="zoom:25%;" />

```bash
git checkout master
git merge experiment
```

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307230039410.png" alt="image-20200307230039410" style="zoom:25%;" />



Take another example:

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307230300755.png" alt="image-20200307230300755" style="zoom:25%;" />

Say you want to merge client into mainline for a release, but you want to hold off on the server changes. You can take C8 and C9 and put them on master.

```bash
# take the client branch, figure out the patches since it diverged from the server branch, and put those patches directly on the master branch.
git rebase --onto master server client
```

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307230611038.png" alt="image-20200307230611038" style="zoom:25%;" />

```bash
git checkout master
git merge client
```

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307230649632.png" alt="image-20200307230649632" style="zoom:25%;" />

```bash
git rebase master server
```

<img src="/Volumes/storage/Documents/md/Git.assets/image-20200307230725673.png" alt="image-20200307230725673" style="zoom:25%;" />

```bash
git branch -d client
git branch -d server
```

## Checkout

Copy files from history or stage to the wd, and to optionally switch branches. 

- If the commit name is provided, the files are copied from commit to staging and wd.
- If commit name is not provided, the files are copied from staging  to wd.

## Commit

```bash
# implicit add all files already existing previous commit.
git commit -a # allows you to skip 'git add'
git commit

# view commit history - see git log
```

## Undoing

```bash
git commit -m 'first commit'
git add forgotten-file
git commit --amend

# unstage file
git add README.md # staged
git status
git reset HEAD README.md # unstaged

# unmodify modified file
git reset HEAD README.md
```

## Diff

```bash
git diff # diff between WD and Staging
git diff --cached # diff between Staging and HEAD

# diff between some-branch and WD
git diff <some-branch>
git diff HEAD

# diff between two commits
git diff first-commit last-commit
```

## Reset

Moves the current branch to another position, and optionally updates the stage and the working directory. It also is used to copy files from the history to the stage without touching the working directory.

```bash
git reset 					# default to HEAD and copies to staging, doesn't touch wd
git reset -- files 	# default to HEAD and copies only the mentioned files into staging, doesn't touch the wd
git reset --head 		# default to HEAD and copies to staging and wd

# HEAD~3 means N-3 commit.
git reset HEAD~3 				# copies history into staging only
git reset HEAD~3 --hard # copies history into staging and wd
git reset HEAD~3 --soft # simply moves the HEAD, doesn't copy to staging or wd
```

## Merge and Rebase

To join two histories, a `merge` or `rebase` is necessary. A `merge` creates a new commit that incorporates changes from other commits. Before merging, the stage must match the current commit. 

## Misc commands

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
git config color.ui true
git config format.pretty oneline
git help config

##### clone a repo
git clone <git url> <desired output directory>
# or connect local to a repo
git remote add origin <server>
# or make sure local repo matches with remote
git pull origin master

#### Create a local repo
$ cd <dir>
git init
git add * 
git commit -m 'some comment'
git commit -a # implicit add for all files already existing in last commit
git status

git checkout HEAD -- files 	# copies last commit to both Staging and WD
git checkout HEAD 					# switches WD and S to last commit in HEAD

#### add
git add -h
git add * 
git add -i # interactive add

##### status
git status # shows which files changed
git status -s # short version

##### Merge A into B
git checkout B
git fetch A
git merge A
# or
git checkout B
git pull A # does the fetch and merge

# Create a branch from master and push the branch to remote
git pull origin master # Make sure local repo matches with remote
git checkout -b <new-branch-name> # create branch and switch to it
git push origin <new-branch-name>

##### create a branch and push to remote
git checkout -b my-branch	# create branch and switch to it
# make changes, git add, git commit -m etc...
git push <remotename> <branchname> # first time
git push -u origin my-branch <can be master> # push the branch to remote for others to see
git branch
git branch -av
git branch -avv
##### rename a branch
git push <remotename> <localbranchname>:<remotebranchname>

##### delete a branch
git branch -d my-branch 		# local
git push <remotename>:<branchname> # remote
git push origin :my-branch 	# remote
git push origin :branch1 :branch2 :branch3

##### pushing code not tracked by repo
$ cd existing-project
git init
git add --all
git commit -m "Initial Commit"
git remote add origin <https://some.github.url.git>
git push -u origin master

##### push code already tracked by repo
$ cd existing-project
git remote set-url origin <https://some.github.url.git>
git push -u origin --all
git push origin --tags

##### pushing local dev branch to master
git push origin dev:master

##### update local with newest commit
git pull # this does fetch and merge

##### merge another branch to your branch
git merge <branch>

##### pulling a different branch to local
git pull origin master:dev

##### difference
# status shows files that are changed, diff shows what inside file has changed.
git diff <source_branch> <target_branch>
# what is staged for next commit, this will not show all changes, only what is staged.
git diff # shows what you've changed but not yet staged
git diff --staged # what you've staged now and that will into next commit
git diff --cached # same as above

# replace some local changes
git checkout -- <filename>

# drop all changes
git fetch origin # fetch latest
git reset --hard origin/master # switch local master to it

# Removing files
# Ignore a file after its already checked in
rm FILENAME # removes from working directory
git rm FILENAME # removes from staging
git commit -f # will remove the file

git rm --cached FILENAME # keep in working directory, remove from staging

##### Moving files
# git figures it out automatically
# manually:
mv oldfile newfile
git rm oldfile
git add newfile
# automatically:
git mv oldfile newfile

# Push tag
git push <remotename> <tagname>
```

## Log

```bash
git log # too long
git log -p -2 # shows last 2 commit details
git log --since=2.weeks # list of commits in last two weeks
# you can specify "2008-01-15", "2 years 1 day 3 minutes ago"
git log --stat
git log --pretty=oneline # compressed log view
git log --pretty=format # shows format help
git log --pretty=format:"%h - %an, %ar : %s"
git log --author=bob # commits by certain author
git log --graph --oneline --decorate --all # decorated
git log --name-status # which files have changed

# commits but not merged
git log --pretty="%h - %s" --author='Junio C Hamano' --since="2008-10-01" --before="2008-11-01" --no-merges -- t
```

## Pull Request

To create a PR, you must have the changes committed to the branch.

https://yangsu.github.io/pull-request-tutorial/

https://hackernoon.com/how-to-git-pr-from-the-command-line-a5b204a57ab1

## File status

File status: `untracked`, `unmodified`, `modified`, `staged`.
* `untracked` is a new file not in previous commit snapshot. `untracked` gets tracked only after add.
* add: moves file from `untracked` to `staged`
* edit: moves file from `unmodified` to `modified`
* add: moves file from `modified` to `staged`
* commit: moves file from `staged` to `unmodified`
* remove: moves file from `unmodified` to `untracked`

## Standard practice

* Two main branches: `origin`, and `develop`
* Multiple supporting branches: `features`, `release-*`, and `hotfix-*`

```bash
##### create feature branch from develop
git checkout -b myfeature develop

##### feature is now stable and ready to move forward
git checkout develop
git merge --no-ff myfeature

##### push develop git push <remote> <branch>
git remote -v 							# what remote branches exist
git branch -avv 						# which remote branches are linked to your local branch
git push origin develop 		# pushes branch develop to remote origin
git push origin dev:master 	# pushes local dev branch to remote  origin's master branch

##### create release branch
git checkout -b release-1.2 develop

##### release is now stable
git checkout master
git merge --no-ff release-1.2
git tag -a 1.2

##### the changes done in release need to be availabel to develop
git checkout develop
git merge --no-ff release-1.2

##### delete release 1.2
git branch -d release-1.2

##### create hotfix
git checkout -b hotfix-1.2.1 master

##### do some work on hotfix
git commit -a -m "some hot fix into 1.2.1"
git checkout master
git merge --no-ff hotfix-1.2.1
git tag -a 1.2.1
git tag 1.2.1 1b2e1d63ff # first 10 of the commit id
git checkout develop
git merge --no-ff hotfix-1.2.1
# delete local branch of hotfix
git branch -d hotfix-1.2.1
# delete remote branch of hotfix
git push origin --delete hotfix-1.2.1
```

## Bash Alias

```
grbi 	- git rebase -i master
git rebase -i --autosquash

gcane - git commit -amend --no-edit
grbc	- git rebase --continue
push	- git push origin `git rev-parse --abbrev-ref HEAD`
gll 	- git log --oneline --decorate
gcf 	- git commit --fixup <commit>

# ~/.gitignore_global can have .gitignore entries
```

## Reference

https://hackernoon.com/git-push-and-pull-tips-and-tricks-7f9163539f02

https://ohshitgit.com/

https://git-scm.com/book/en/v2

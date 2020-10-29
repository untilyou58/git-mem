# Intermediate Git for working adults dedicated to those who have completed the introductory book

This artice has wrote by `yamamoto7` [入門書を終えた人に捧げる、社会人のためのGit中級編](https://qiita.com/yamamoto7/items/fe15a1e7e360b4015fae)

I've summarized the commands and useful settings that I often used to actually work in a company.
I think that the introduction to Git is saturated, but I hope you can see it as a little advanced version.

Environment:

- Huge number of files and lines
- Multiple projects are often in progress at the same time, and branches are often moved to answer questions.
- Multiple character codes are mixed in the project (Shift-JIS and UTF-8)

## Command

### Basic command writing

```cmd
git clone <branch name> <directory name> #Clone by specifying the destination directory name 
git pull #pull. Type -u, remote name, or branch name as needed 
git diff # View diffs 
git diff master HEAD # Compare current state with master 
git checkout -b <branch name> # Create a new branch Check out
```

### Save the file in the middle of work

I think that it will be an option such as stash, but in my case I will `commit and save the work status`.
Since it is a commit, the work status is not actually saved, but here the word save is selected.
The commit message can be anything, but "WIP" which means "working"

```cmd
git add <file you want to save>
git commit -m WIP
```

If you often move branches, it is recommended to save the work by commit.
It's not easy to evacuate once and resume work after a week or two, so it's easier to organize by committing.

When I do a little additional work, I feel like doing an additional WIP commit.

### Restore the saved commit

Since it is not stash, it is an image of canceling WIP commit and returning to workspace rather than restoring.

The `reset` command cancels the commit, but if you want to use it as a substitute for stash, you should use the `--mixed` command.
`--mixed` has options such as removing commits and reverting file changes to workspace.

```cmd
git reset --mixed <commit hash>
# git reset --mixed HEAD ^ (go back one) 
# etc.
```

### Add

Any method can be used here, and a normal add is sufficient, but I will introduce the method I am doing.

The add command `-i` allows you to add interactively with the option. Please check this [official website](https://git-scm.com/book/en/v2) as it will take a long time to use it in detail

```cmd
git add -i
```

## Settings

### Fast forward setting

It's relatively important

```
[ merge] 
    ff =  false 
[ pull] 
    ff = only
```

If you set ff = false, it will always make a merge commit when you merge.
The merge performed on github is attached by default.

On the contrary, when pulling, absolutely accept only fast forward.
I don't think it's necessary to be fast forward.
You might think that you don't need this if you're operating normally, but that's right. For psychological safety.

### Frequently used command alias

This is my favorite. Immediate access to familiar commands.
I would like to introduce the ones that I especially use.

```~/.bashrc
# Abbreviated status 
alias s='git status -s'
# Graph display of last 20 lines 
alias l='git log --pretty=oneline -n 20 --graph --abbrev-commit'
# Conv Respect settings grep
alias gg='git grep --textconv'
```

```
[alias]
    di = !"d() { git diff --patch-with-stat HEAD~$1; }; git diff-index --quiet HEAD -- || clear; d"

    # Latest commit and current status show the differences between
    d = !"git diff-index --quiet HEAD -- || clear; git --no-pager diff --patch-with-stat"

    # All add to Commit
    ca = !git add -A && git commit -av

    # Find branch containing a specific commit (find branch) 
    fb = "!f() { git branch -a --contains $1; }; f"

    # Find log in source code (find by code)
    fc = "!f() { git log --pretty=format:'%C(yellow)%h  %Cblue%ad  %Creset%s%Cgreen  [%cn] %Cred%d' --decorate --date=short -S$1; }; f"

    # commit message search log (find by message) 
    fm = "!f() { git log --pretty=format:'%C(yellow)%h  %Cblue%ad  %Creset%s%Cgreen  [%cn] %Cred%d' --decorate --date=short --grep=$1; }; f"

    # delete merged branch merged into master
    dm = "!git branch --merged | grep -v '\\*' | xargs -n 1 git branch -d"
```

### Settings to prevent garbled characters

Since I was in charge of a project in which Shift-JIS and UTF-8 were mixed, I made this setting.
Once set, commands such as diff can be used without garbled characters . It is a trade-off with execution speed.

```
[diff "mixed"] # Prepare your own namespace for mixing 
    textconv = nkf -w8
```

```
# Select the target extension
*.html diff=mixed
*.js diff=mixed
*.css diff=mixed
```

Although it is not handled this time, you can also set the handling of Excel and binary files with textconv. ( [Reference](https://qiita.com/takedakn/items/660d17fe10ede8441ee6) ) After making
this setting, you can execute the command with respect for textconv by adding the following options.

```
$ git diff --textconv
$ git show --textconv
$ git grep --textconv <検索したい語句> <検索をかけたいディレクトリ>
#プロジェクトが巨大なため、検索速度を速めるためにディレクトリまで指定しています。
```

### Signed commit

When you commit, such a mark will be added on GitHub etc.

It's not required, but it looks like an official mark and is cool.

```
[ user] 
    signingkey = <each key>
 [ commit] 
    gpgsign =  true
```

In addition to this setting, you need to install a GPG client that suits your environment.
By default, commits made on github are signed

## Case Studies

### Please review (when requested)

When I was asked to review, I wrote down how to go to the target branch

```
# First , save your work
$ git add -i # Add interactively using -i
$ git commit -m WIP

# Get the status of origin (1st time required, 2nd time and later can be skipped)
$ git checkout master
$ git pull origin master

# Go to the branch to be reviewed
$ git checkout <branch name to be reviewed>
$ git pull origin <branch name> # Only the second and subsequent times in the same branch

# Test or diff check Or
```

### I'm getting an error before I know it, but I don't know the cause.

Most problems can be solved visually or by standard output, but it is used when you really do not understand .
Basically, reset and checkout are repeated and specified

If you have too much history and it seems like it will take a long time, use the command `bisect`.

### Different with reset and checkout

```
# There seems to be some, but not so much. But it's safe to have a way to identify the error within yourself.
# The finer the commit, the easier it is to identify in detail with git.
# It will be easier if you devise it, but to avoid confusion, I will practice it with only simple commands.
# Various things can be done by combining the following commands.

# ※※※Please save the work in progress as it will disappear※※※

# 1: Find an era that will be normal
$ git log
$ git reset --hard <commit hash>

# If the error still continues, return to 1
# 2: If normal, look for the original commit hash to return to the original state 
$ git reflog
# （Example output）
# 955f116 (HEAD -> master, origin/master, origin/HEAD) HEAD@{0}: reset: moving to HEAD^
# 55ed0ee HEAD@{1}: commit: aaaa

# 3: Reset to the original state
$ git reset --hard <commit hash> # In the above output example, commit hash returns to 55ed0ee

# 4: 1 and identifies the error commit

# 5: Once identified, find out which file is the cause (use reset --mixed or checkout <filename> to find it)
```

### Dichotomy with bisect to identify error commit

[This](https://qiita.com/usamik26/items/cce867b3b139ea5568a6) is easy to understand
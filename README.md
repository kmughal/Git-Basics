# Git usefull tips

This is a simple guide that helps newbies to get the most out of git.

- [Git fundamentals](#git-fundamentals)
- [Plumbing](#plumbing)
- [Viewing the history](#viewing-the-history)
- [Viweing diffs](#viweing-diffs)
- [Show commits](#show-commits)
- [Staging and Indexing](#staging-and-indexing)
- [Commits verification](#commits-verification)
- [Writing a good History that leavs a trail](#writing-a-good-history-that-leavs-a-trail)
- [Resolving conflicts](#resolving-conflicts)
- [Fixing a mistake](#fixing-a-mistake)
- [Finding a bad commit](#finding-a-bad-commit)

## Git fundamentals

Git uses internally commit objects that contains metadata such as author timestamp and a refferance to a parent commit, each commit represent a snapshot of a working directory and it is stored as tree object, a tree contains blobs the objects that represent file changes also tree may contains other tree(s)

```git
Commit: contains author, timestamp, refferance to a parent commit
    |
    + Tree: this represent the directories
        |
        + Blob: this represent the files
```

You can directly point to a commit tree or blob using unique ID known which consists the calculated SHA-1 hash of the object content, the IDs are not best way to reffer to this object so git uses other type of refferances:

- **tag**: fixed refferance to a specific commit and never changes
- **branch**: is refferance to the last commit in line of history
- **head**: special refferance managed by git, used to keep track of the branch or commit whose shanpshot to whats in the working directory

The best way to demostrate how this looks like is via the lower level plumbing command documented bellow

### Plumbing

Plumbing is lower level command to explore the snapshot, for example the command `git cat-file -p head` will show the raw commit with all its metadata including trees and blobs, you can use the unique SHA-1 to explore the trees and blobs further using the same command `git cat-file -p 9f3322`

## Viewing the history

In the command line the best way to visually view the history isuing the command bellow:

```git
git log --pretty='%C(red)%h%Creset | %C(yellow)%d%Creset %s %C(green)(%cr)%Creset %C(cyan)[%an]%Creset' --graph --all
```

> it is also a good practice to create an alias for it call `lga`

## Viweing diffs

You can view diffs in git using the commad `git diff` this will print all the data diff related to the index and specific commits, you can also manipulate this data for example `-s` flag is used for scrollable content and then to scroll using `j` and `k` keys, in case of documentation it is nice to view word diff instead of line diff this can be done using the flag `--word-diff` and to increase the lines arround the changes to get more context you can use the flag `--unified=10` this would show 10 lines before and after the change instead of the default 3 lines, use the flag `--patiance` to get more accourate diff or `--historgram` also to get summary of the changes use the flag `--stat`

You can view the diff of 2 commits using the double dote notation `git diff head..head~1` and apply any of the flags above

To compare a specific file just add `:filename` in this way `git diff head:README.md..head~1:README.md`

## Show commits

You can see a commit using the command `git show head~2`, you can pass any valid refferance, the flag `--no-patch` will show only the metadata, to get summary of the changes use the flag `--stat`

## Staging and Indexing

In git staging or indexing is used to take control of what will be commited

### Stage all the changes

This is a **bad practice** and needs to be avoided, because you do not elect what goes into the commit and often lead to unwanted changes goes with the commit

```sh
git add .
git commit -m "feat(some-scope): subject and description"
```

### Stage multiple files

Staging is greate practice because you choose what goes into your commit

```sh
git add file1 file2 file3
git commit -m "feat(some-scope): subject and description"
```

### Stage a part of a file

This is powerfull feature of git allow you to stage part of a file

`Example`

See the code bellow `function test` is the function that needs to be commited the other code should not be commited as it was written to help testing the functionality

```js
function test(text,condition)
{
    if(condition()) console.info(text," succeeded");
    else console.warn(text," failed");
}

function checkIfTestWork()
{
    var x = "some string";
    test("if x was not global",function(){ return x; });
    test("if boolean + object is a string",function(){
        var bool = true;
        var obj = {};
        return typeof(bool+obj)=="string";
    });
}
```

In this case we only ant to stage part or patch of this file

```sh
git add main.js -p
```

The command line will be showing `hunks` and asking `Stage this hunk [y,n,q,a,d,/,s,e,?]?`

- `y` yes include this hunk
- `n` no skip this one
- `q` quite all
- `a` add all
- `d` skip rest of the file
- `/` search for hunks using regex
- `s` split to smaller hunks
- `e` edit hunk
- `?` help

Also you ca use

- `j` navigate to next hunk
- `k` navigate to previous hunk

### Interactive staging

This must be your best friend, because you can descide about all the changes in your work and decide what to add entirly and what to add partially

```sh
git add -i
```

This will allow you to chose the files that you want to stage and decide what to do with it in interactive way, once your done you can always type `q` for quite

## Commits verification

Once your about to commit, it is time to run series of verefication to ensure that the commit is `a good commit known as ACID commit`

### ACID commit

> `Atomic` self contained: no symantic changes must be splited (for example rename function should be commited as well as renaming all its usage in all the project), also the commit must be one logical change, uslally commit must be between 2 to 5 files
>
> `Consistent` Commit should leave the code base in cosistent state (code must complile and passes the tests)
>
> `Incremental` Commits should be ordered logically, reflecting the evolution and explain the process that a feature went through
>
> `Documented` Commit message should describe the meaning and the role of a change, using our commit format

### Diff

This is very usefull command that help you check the commits

- It shows you the changes using `git diff`
- It shows invalid whitespace using `git diff --check`

> `Att:` if you find white spaces applied to your local commits, you can quickly remove them using `git rebase Head~some-number --whitespace=fix`

### Check if the code works

> The main question here to ask is how to test the code that you staged and commited but not the other dirty files ?

The answer is the `stash`, the stash is temporal storage for unfinished work

> stash = dirty files + staged files

`Example`

In the example bellow we added 2 functions `add` and `subs` but we only commited `add` as subs is yet to be compeletd and commited in diffrent commit

```js
function main(){
    console.info('application started...');
}

//added code
function add(x,y){
    return x + y;
}

function subs(x,y){
    //code to be added
}
```

So now it is time to test if what has been commited is working as it should

```sh
git stash save --keep-index --include-untracked "subs functionality in progress"
```

Now your work space is clean and contains only the changes that you are about to commit, so you can build run and verify that it is all working as it should

> After you verify and you commit you can then call `git stash pop` and your changes will be restored back as well as they will be removed from the temp storage

## Writing a good History that leavs a trail

One of the most important is to leave a trail in the history so that it looks as a logical series of steps that document the software evolution, take a look at the example bellow:

Bellow is a **good trail** in a feature branch, which contains 3 `ACID` commits:

[ Refactoring ] <- [Add acceptance test] <- [ Feature ]

- Refactoring: this is the change that leaves a room for the feature to be alive
- Add acceptance test: this is an **ignored** test that will be used to test the feature, it is ignore so that the commit is `ACID`
- Feature: is the commit that actualy ads the feature

A **bad trail** example would be something like that:

[ Refactoring ] <- [ Rename a file ] <- [Add acceptance test] <- [Delete a file] <- [ Feature ]

Luckly in git you can make a bad history like the one above to a beter one

### Change a bad history to a good trail history

One of the ways to clean up local repository so that it will have better trail history is using `interactive rebase`

#### Public vs Private history

When working you do few `ACIS` commits and you only push the chages once you are happy with the code and once everything is verified, before the push the history called `Private` while after the push it becomes `Public`.

> **You are allowed to only change the private history that exist only at your local**

#### Interactive rebase

This will allow you to perform action such as 'reorder, change, or merge to commits'

In our example, we would want to change the head~4 range, in this way:

`git rebase -i head~4`

3 pick Rename function
2 squash Delete file
1 reword Acceptance test
0 edit Feature

This will make git to process the given range and:

- the first stop will be the squash allowing us to change the commit message to "Refactoring"

- the second stop will be the reword, and we can give it the commit message "Add acceptance test"

- finally we can stop to edit the content of a commit (maybe reoriding functions or removing comments or unwanted empty lines or spaces)

The history would like like this:

[ Refactoring ] <- [Add acceptance test] <- [ Feature ]

> Once your done you can stage the changes `git add .`
>
> and apply it as last commit `git commit --ammed -C head`
>
> and then we can continue to finish the rebase `git rebase --continue`

### Branching and Merging

Branches in git are so powerfull unlike other source controlls git makes it easy to manage branching and Merging

In git there are 2 type of branches log running branch (like master release and develop) and topic branches like (hotfix feature-branch), also as mentioned before branches can be private and public

Regardless of the branche type in git you create the branch in the same way `git checkout -b feature-branch`

#### Merge fast forword

If the merged branch reffer to the head of the destination branch

See the example bellow:

```git
[a] <- [b] <- [c]:master <------ [d] <- [e]:feature
```

feature is reachable from the master and feature point to the last commit of the master, in this case we can do merge fast forword simply by using `git merge branch-name`

If that's not the case, feature not pointing to the last commit from master

```git
[a] <- [b] <- [c]:master
          <- [d] <- [e]:feature
```

In order to do fast forword it is neccesary to use `git rebase master feature` and then you can apply the merge `git merge branch-name`

#### Merge true merge

If you want to ensure that you make a true merge simply append the flag `--no-ff` in this way `git merge branch-name --no-ff`

> Here is the golden rule:
>
> Fast forword merge must be used when the merged branch is private
>
> True merge is used when merging public branches

#### Cherry pick

This is another powerful feature of git that allow you to merge single commit or range of commits from a branch to another, unlick merge you do not have to merge all commits but you can pick the onces you want to merge

Here is an example:

```git
[a] <- [b] <- [c]:master
         <- [d] <- [e]:feature
```

you can append the commit [e] to the branch master simply by this command `git cherry-pick feature`

if you want to specify a range you can use dot dot syntax this way `git cherry-pick feature~3..feature` this will append [d] and [e] to master

## Resolving conflicts

When merging rebasing or cherry-picking git looks at every patch from each commit and if there will be a line that has been changed in both sides git will just stop and wait for us to resolve the conflict

In this case you can abort the merge using `git merge --abort` this will revert your working directory to the clean head

But if you need to merge and resolve the conflict just do `git status` to see the list of the changed files in conflict, open each file and resolve it using this steps:

- Open the file and manually resolve the conflict

Git uses revsion control notation to indeicate the conflict on the files you will find the conflicted lines marked in this way

```git
<<<<<<< ours
console.log("hello")
=======
console.log("hello world")
>>>>>>> thiers
```

important to note here that the line `ours` is the line from the commit that has been marged to and refferenced by `head`, while `thiers` and this is from the commit that has been merged and reffernced as `merge_head`

At this point it maybe unclear which line to use, and here is where git shine

- Check the commits that include the conflicting files

`git log --merge --oneline` this command should show the 2 commits that causes the conflicting files reading the message and understanding the context is the key to the conflict resolution

If this does not help you may want to look at the commit before the one that has the conflicted files, in git you use `git merge-base head merge_head` to find this commit

If also this does not give us any clue, there is another way to find out

- 3 ways merging to include the common ancestor line into the conflict

In git you can do that for each file `git checkout --conflict=diff3 main.js` and this will produce something like that:

```git
<<<<<<< ours
console.log("hello")
||||||| base
console.log("hello")
=======
console.log("hello world")
>>>>>>> thiers
```

- Finally if you decide you can use either `git checkout --ours main.js` or `git checkout --thier main.js` or you can modify the file manulally to combine the 2 changes, **but never introduce new change that does not exist in both side this is known as evil merge**

### RERERE

RERERE stands for Resuse Recorded Resolution, it is a useful feature of git, that can be very helpful and time saver

Imagine that you perform a test merge to check if your code will work but you face a conflict and you resolve it, after you are done with your test merge you will want to fix what needs to be fixed and apply the real merge, but again the conflict will happen `rerere` allows you to reuse the same recorded automatically which is a real time saver

To use this feature you will have to enable it using `git config --global rerere.enable true` and then you can apply the merge `git merge feature` after resolving the merge the resolution will be recorded and in the future merge it will be automatically apply but the file won't be staged so that you can have a chance to review it

## Fixing a mistake

Git is more forgiven than any other source control out there is more human as it understands that we are all humans and make mistakes, so as soon as you have not share it in public you can change things up

### Ammend last commit

You can ammend the last commit by using `git commit --ammend` and if you want to add file or make changes just stage it and use the same command, if you want to use the same commit message append `-C` in this way `git commit --ammend -C`

### Git rid of a commit

It is common that after a a commit, you may change your mind and say that this was not a good idea, with Git that's not a problem as soon as it is private commit you can remove it using `git reset --hard head~1` note that `--hard` will update the index as well as the working copy so **any local changes will be lost** and so you must be carfull when you use this command

### Squash to last commits

This can be done quickly without a rebase using `git reset --soft head~1` this will move the head only index and working directory will remain the same so we just need to do `git commit --ammend -C`

### Revert to a commit and keeping the files

This can be achieved with `--mixed` which move head and update only the index, this is usefull if you change your mind about commits and want to do some major changes and commit them in diffrent way

### Ammend a far away commit

Sometimes the commit you want to change is not the nearest one, the easiest way to do that is to:

- create a new commit that has the changes you want to do
- squash it to the commit you want to ammend

To do that you can use the magic feature of git `auto-squashing`, to do that follow this steps:

- create a new commit with message that start with this string `fixup!` followed by **the same commit message like the commit you are targeting** the command may look like that `git commit -m "fixup! feat(some-feature): some subject ..."`

- after start the interactive rebase starting from first commit `git rebase -i --root --autosquash` and leave git to do its magic

- save the rebase file and exit

### Undo action

In git there is a journal that records history called `reflog` there is one for the head and there is also `reflog` for each branch, and thanks to this journal we can undo our actions using this command `git reset --hard master@{1}`

### Recovering a commit

Again the `reflog` records all the actions so we can retrive commits from there as soon as they have not been removed by the garbage collector of git this between 30 to 90 days depanding if its unreachable or rechable commit

First we need to find the reflog entry to the commit we want to retrive we can do that using `git log -grep=feat(some-feat) --walk-reflogs`

After we find the reflog entry we can do `git reset` or `git cherry-pick` as exaplained above here is an example of how to do that `git reset --mixed feature@{4}`

## Finding a bad commit

Git has very intersting feature that allows you to find a bad commit automatically in a given range the command is simple `git bisect start head head~10` here we are saying that the head is the bad commit and we ask git to check from the last 10 commits to find when the problem start happen, then we instruct git on how to find the bad commit `git bisect run npm start` in this example we run node start command that may exit with `status` 1 if it is a bad commit or `status 0` if it is a good commit, git will do its own thing and get back to us with the first bad commit pointing to the problem

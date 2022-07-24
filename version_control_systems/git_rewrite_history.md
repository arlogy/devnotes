# Rewrite Git History Locally or After Push

## Introduction

After mistakenly `git push`ing to a repository, for example when private details
such as name or email have been shared, or a commit performed under an
inappropriate username for those with mutiple Git accounts, one might want or
need to rewrite the history of the repository to correct one's mistakes.

## Warning (team vs solo)

However, rewriting change history can cause others to lose their work unless
done carefully (e.g. before the changes to be rewritten are `git pushed` ed). So
this is not something to be done out of the blue when collaborating on a
project. Of course, consequences are different when working solo, because the
developer would not need to worry about throwing others' effort and contribution
away, thus getting them frustrated.

## Let's start step by step

In the steps outlined below, we will see how to rewrite the history of a Git
repository with a single branch (`master` or [`main`](https://github.com/github/renaming)).
Note that branches and tags are separate concepts in Git, so if you've rewritten
branches, you might want to delete and reshare some tags as well.

Besides, when working with multiple branches, you should also rewrite the
branches created from the branch you are about to rewrite, so that they can take
into account the new history. In this case, first read about the steps available
below, then [look for more information on the internet](https://www.google.com/search?q=git+rebase+all+child+branches).

### Step 0 (preparation)

- If this is your first attempt to rewrite the history of a Git repository,
please consider reading the documentation first: [Git Tools - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History).
You can stop right before the *Reordering Commits* section, but all the content
is actually interesting.
- Create a local copy of the repository (including the `.git` directory). Such
a temporary backup of the repository is not necessary whatsoever, but it will be
useful if you want to replicate exact commit dates from the old change history
into the new one.
- Make sure that Git is configured to use appropriate user email and name. These
will be used when `git commit`ting changes to the repository unless otherwise
set per commit.

### Step 1 (choose which part of the history to rewrite)

- `git rebase -i --root` or `git rebase -i HEAD~3` for example.
    - `--root` enables us to rewrite the history starting at the first commit.
    See sources below.
        - https://git-scm.com/docs/git-rebase#Documentation/git-rebase.txt---root
        - https://stackoverflow.com/questions/22992543/how-do-i-git-rebase-the-first-commit/23000315#23000315
        - https://stackoverflow.com/questions/2119480/edit-the-root-commit-in-git/14630424#14630424
    - `HEAD~3` allows us to rewrite the history of the last three commits only.
- An editor will open to allow you to choose the commits to edit. Replace `pick`
with `edit` in the open editor for each commit you want to edit. Save your
changes and close the editor.
- Now you are ready to start rewriting the history of the repository. Your
command line prompt will also show some basic information to help you track your
progress during rewrite: e.g. `<path_to_project_repository> (master|REBASE 1/3)`.

### Step 2 (rewrite the history)

- Make changes as desired. For example, you can change the commit message with
`git commit --amend`. Or if you prefer, you can edit and/or remove files prior
to `git commit`. In any case, be sure that author date and commit date are set
to the same expected value. You can run the following command for this purpose:
`GIT_COMMITTER_DATE="..." git commit --amend --date="..." --no-edit`.
    - The `GIT_COMMITTER_DATE` environment variable will set the author date
    while the `--date` option will set the commit date. It seems that the author
    date is the one shown online on github.com for each commit, at
    https://github.com/<username>/<repository_name>/commits for example.
    - RFC 2822 is a possible date format, e.g. `Thu, 07 Apr 2005 22:13:13 +0200`.
    If you want to use the exact date values you used before starting to rewrite
    the history, open a command line prompt on the local backup you created
    earlier for the repository and run `git log --pretty=fuller > tmp.txt` or
    `git log --pretty=fuller HEAD~3..HEAD > tmp.txt`. In the temporary `tmp.txt`
    file, you will see (among other things and in reverse order compared to the
    interactive `git rebase -i`) the author date and the commit date of each
    commit.
    - See sources below.
        - https://git-scm.com/docs/git-commit/#_commit_information
        - https://git-scm.com/docs/git-commit/#_date_formats
        - https://stackoverflow.com/questions/454734/how-can-one-change-the-timestamp-of-an-old-commit-in-git/5017265#5017265
        - https://stackoverflow.com/questions/19742345/what-is-the-format-for-date-parameter-of-git-commit/19742762#19742762
- Run `git rebase --continue`.
    - Now you can rewrite the next commit as shown in your command line prompt:
    e.g. `<path_to_project_repository> (master|REBASE 2/3)`. We just need to
    restart at step 2 above until `git rebase` finishes. Also note that you
    might need to resolve merge conflicts in case Git fails to apply your
    changes to the next commit.

### Step 3 (share the new history)

- `git push --force origin master`
    - You might also want to use the `--force-with-lease` option instead of
    `--force`. But this security wall is likely to be unuseful for the solo
    developer.
- End

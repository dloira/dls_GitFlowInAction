# Git Flow In Action with multi release candidates

Here, you could find the principles to use Git Flow when the teams are pushed to work with multiple release candidate branches at the same time. Day by day, the source code maturity differs from one version to another, in terms of when them could be promoted to Production environment; however, it is not so weird to be under water when Production bugs arise and at the same time QA team is checking the following version on Stage and the next one on Test environments.

Git Flow shows us the way considering the key moment to branch off a new release branch from develop is when develop (almost) reflects the desired state of the new release. It is one of the most relevant golden rules for getting our repositories under control; unfortunately, Git Flow doc references do not get into when this rule needs to be broken.

## Getting Started

Git Flow is a git branching model developed initially by Vincent Driessen in this [blog post](https://nvie.com/posts/a-successful-git-branching-model/). 

You really should go read the post to learn about it, but the high-level idea is that there are two long-lived branches **master** (which is always what is deployed in production) and **develop** (which is the latest and greatest development code). To make new features, **develop** is branched off of into **feature branches**. To prep a new release for deployment, once the new features were merged in **develop**, a **release candidate** is branched off of **develop**, which will then be merged to master. To do a hotfix on production, **master** is branched off of into **hotfix** for bugfixing, which will be merged back into **master**.

> **_NOTE:_**  The git repository settings has to be enabled the **Fast-forward merge** (No merge commits are created and all merges are fast-forwarded, which means that merging is only allowed if the branch could be fast-forwarded. When fast-forward merge is not possible, the user is given the option to rebase.). 

> When **Merge commit** is enabled instead, a merge commit is created for every merge and it will not help for branching as it will be explained below.

## Walk-through

Once the basic concepts are out there, it will go through the most relevant use cases:

### Building new features

**Feature branch** (f_1) has to be branched off of **develop** (not **master**); once coding is ready for testing (end to end, loading, regression and so on) the **feature branch** (f_1) is merged into **develop** by squash for compacting the whole commmits chain into an unique commit.

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![Feature_1](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_FEATURE-BRANCH_1.png)

Git bash snipped hereunder.

```
1.- Create feature branch from develop

$ git checkout develop
$ git pull --prune
$ git checkout -b f_1 develop
$ git push --set-upstream origin f_1

... coding the feature to be built ...

2.- Merge feature branch into develop with squash

$ git checkout f_1
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git merge --squash origin/f_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'f_1' into 'develop'"
$ git push --force

```

### Building new urgent features or develop with unstable commit inside 

**Feature branch** (f_1) has to be branched off of **develop** (not **master**); once coding is ready for testing (end to end, loading, regression and so on) the **feature branch** (f_1) can not be merged into **develop** because it is mandatory to release the feature as soon as possible and it not an option to wait for finishing the testing of current commits on **develop** branch (or maybe, there is a commit under quality threshold by mistake).

When this scenario arrives, **release candidate** (rc_1) has to be branched off of **develop** from the commit behind of unstable commit. Next step, **feature branch** (f_1) is merged into **release candidate** (rc_1) by squash for compacting the whole commmits chain into an unique commit. Hereinafter, **feature branch** (f_1) is not needed never again.

Once feature bugfixing is ready is time to merge **release candidate** (rc_1) into **master**; be very careful with not to squash, otherwise **master** will be updated with only one commit, breaking the history commits chain shared between **develop** and **release candidate** (rc_1) but not with **master**.

Finally, **develop** has to be rebased from **master** to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![Feature_2](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_FEATURE-BRANCH_2.png)

Git bash snipped hereunder.

```
1.- Create feature branch from develop

$ git checkout develop
$ git pull --prune
$ git checkout -b f_1 develop
$ git push --set-upstream origin f_1

... coding the feature to be built ...

2.- Create release candidate branch from develop (commit behind of unstable commit)

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_1 e820073d
$ git push --set-upstream origin rc_1

3.- Merge feature branch into release candidate with squash

$ git checkout f_1
$ git pull --prune
$ git checkout rc_1
$ git pull --prune
$ git merge --squash origin/f_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'f_1' into 'rc_1'"
$ git push --force

... bugfixing ...

4.- Merge release candidate into master

$ git checkout rc_1
$ git pull --prune
$ git checkout master
$ git pull --prune
$ git merge origin/rc_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'rc_1' into 'master'"
$ git push --force

5.- Rebase develop from master

$ git checkout master
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/master

... resolve the conflicts ...

$ git push --force

```

### Testing new feature on Stage and promoting to Production 

**Release candidate** (rc_1) has to be branched off of **develop** (not **master**). Maybe the testing takes time to be ready and several commits could be needed to push for bugfixing; the best practices suggest to sync **develop** continously for avoiding a huge sync at the end. These day by day syncs have to done taking in mind that **develop** was including new commmits coming from other features branches once the **release candidate** was created; for this reason, it is needed to rebase **develop** from **release candidate** (rc_1) in order to sync the bugfinxing commits in the proper history commits chain.

Once bugfixing is over, the next step is for merging **release candidate** (rc_1) into **master**; be very careful with not to squash, otherwise **master** will be updated with only one commit, breaking the history commits chain shared between **develop** and **release candidate** (rc_1) but not with **master**.

Finally, **develop** has to be rebased from **master** to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![ReleaseCandidate_1](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_RELEASECANDIDATE-BRANCH_1.png)

Git bash snipped hereunder.

```
1.- Create release candidate branch from develop

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_1 develop
$ git push --set-upstream origin rc_1

... bugfixing ...

2.- Rebase develop from release candidate to sync both and avoid huge conflicts later on (repeat this steps as many times as needed)

$ git checkout rc_1
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/rc_1

... resolve the conflicts ...

$ git push --force

... bugfixing ...

3.- Merge release candidate into master

$ git checkout rc_1
$ git pull --prune
$ git checkout master
$ git pull --prune
$ git merge origin/rc_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'rc_1' into 'master'"
$ git push --force

4.- Rebase develop from master

$ git checkout master
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/master

... resolve the conflicts ...

$ git push --force

```

### Testing new feature on Stage and promoting to Production, when another new feature is being also tested on Test at the same time 

There are two **release candidate** branches alive at the same time: rc_1 for testing a new feature on Stage (it was tested in Test previously) and rc_2 for testing another new feature on Test; both were branched off of **develop** (not **master**) when the features were ready from their feature branches (first, rc_1 from an older commit and rc_2 was next one from a recent commit). 

Once the bugfixing in rc_1 is over, it is time to merge it into **master** as it was explained before; however, please advise to execute some steps before, otherwise the historical commits chain will be broken between **develop** and **master**. The first step is to close all **release candidates** created from commits after the one which rc_1 was built from; morevoer, this closing steps has to be executed one by one and in reverse ordering followed to their creation (first, release candidate branch created from the most recen commit).

These release candidates closings have to done taking in mind that **develop** was including new commmits coming from other features branches once the **release candidate** were created; for this reason, it is needed to rebase **develop** from **release candidate** (rc_2) in order to sync the bugfinxing commits in the proper history commits chain; please advise the SHA hashes overwriting for further existing commits on **develop** (maybe there are more than two **release candidate** branches alive, in these case, it should be needed to rebase **develop** from rc_5, next from rc_4, next from rc_3 and finally from rc_2; please, remember the hashes overwriting with every **develop** rebase).

Once bugfixing is over in rc_1 and every **release candidate** closed, the next step is for merging **release candidate** (rc_1) into **master**; be very careful with not to squash, otherwise **master** will be updated with only one commit, breaking the history commits chain shared between **develop** and **release candidate** (rc_1) but not with **master**.

Next one, **develop** has to be rebased from **master** to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

Finally, new **release candidate** branched off of **develop** from the last commit rebased from the previous **release candidate** because the testing on Test has to be continued (when **develop** was rebased from rc_2 to close it, a bunch of commits were sync from rc_2; however the testing on Test was still on track and it is needed to keep it with the same scope; for this reason, the new **release candidate** branch has to come from the last commit sync in **develop** from rc_2).

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![ReleaseCandidate_2](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_RELEASECANDIDATE-BRANCH_2.png)

Git bash snipped hereunder.

```
1.- Create release candidate branch from develop for feature to test on Stage

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_1 develop
$ git push --set-upstream origin rc_1

... bugfixing ...

2.- Create release candidate branch from develop for feature to test on Test

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_2 develop
$ git push --set-upstream origin rc_2

... bugfixing ...

3.- Rebase develop from release candidate rc_2 to sync both before to merge rc_1 to master (repeat this steps as many times as release candidate branches exist)

$ git checkout rc_2
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/rc_2

... resolve the conflicts ...

$ git push --force

... bugfixing ...

4.- Merge release candidate rc_1 into master

$ git checkout rc_1
$ git pull --prune
$ git checkout master
$ git pull --prune
$ git merge origin/rc_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'rc_1' into 'master'"
$ git push --force

5.- Rebase develop from master (theorically, this step it is not needed, however it would be great to be executed anyway)

$ git checkout master
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/master

... resolve the conflicts ...

$ git push --force

6.- Create release candidate branch from develop for feature to test on Stage

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_3 25b0747a
$ git push --set-upstream origin rc_3

```

### Bug on Production

**Hot fix** (hf_1) has to be branched off of **master** (not **develop**). Once fix is ready, the **hot fix** (hf_1) is merged into **master** by squash for compacting the whole bugfixing commmits chain into an unique commit.

Finally, **develop** has to be rebased from **master** to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![HotFix_1](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_HOTFIX-BRANCH_1.png)

Git bash snipped hereunder.

```
1.- Create hot fix branch from master

$ git checkout master
$ git pull --prune
$ git checkout -b hf_1 master
$ git push --set-upstream origin hf_1

... bugfixing ...

2.- Merge hot fix branch hf_1 into master

$ git checkout hf_1
$ git pull --prune
$ git checkout master
$ git pull --prune
$ git merge --squash origin/hf_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'hf_1' into 'master'"
$ git push --force

3.- Rebase develop from master

$ git checkout master
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/master

... resolve the conflicts ...

$ git push --force

```

### Bug on Production, when a new feature is being also tested on Test at the same time 

**Hot fix** (hf_1) has to be branched off of **master** (not **develop**). Once fix is ready, the **hot fix** (hf_1) is merged into **master** by squash for compacting the whole bugfixing commmits chain into an unique commit.

Next step should be to rebase **develop** from **master**; however, please advise to execute some steps before, otherwise the historical commits chain will be broken between branches. The first step is to close all **release candidates** alive going one by one and in reverse ordering followed to their creation (first, release candidate branch created from the most recen commit).

These release candidates closings have to done taking in mind that **develop** was including new commmits coming from other features branches once the **release candidate** were created; for this reason, it is needed to rebase **develop** from **release candidate** (rc_1) in order to sync the bugfinxing commits in the proper history commits chain; please advise the SHA hashes overwriting for further existing commits on **develop** (maybe there are more than one **release candidate** branch alive, in these case, it should be needed to rebase **develop** from rc_4, next from rc_3, next from rc_2 and finally from rc_1; please, remember the hashes overwriting with every **develop** rebase).

Now, **develop** has to be rebased from **master** to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

Finally, new **release candidate** (rc_2)branched off of **develop** from the last commit rebased from the previous **release candidate** because the testing on Test has to be continued (when **develop** was rebased from rc_1 to close it, a bunch of commits were sync from rc_1; however the testing on Test was still on track and it is needed to keep it with the same scope; for this reason, the new **release candidate** branch has to come from the last commit sync in **develop** from rc_1).

Please, find below one example for better understanding (balls are the commits, SHA hashes are on top and the number inside shows when the commits was made on each branch).

![HotFix_2](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_HOTFIX-BRANCH_2.png)

Git bash snipped hereunder.

```
1.- Create hot fix branch from master

$ git checkout master
$ git pull --prune
$ git checkout -b hf_1 master
$ git push --set-upstream origin hf_1

... bugfixing ...

2.- Merge hot fix branch hf_1 into master

$ git checkout hf_1
$ git pull --prune
$ git checkout master
$ git pull --prune
$ git merge --squash origin/hf_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'hf_1' into 'master'"
$ git push --force

3.- Rebase develop from release candidate rc_1

$ git checkout rc_1
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/rc_1

... resolve the conflicts ...

$ git push --force

4.- Rebase develop from master

$ git checkout master
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/master

... resolve the conflicts ...

$ git push --force

5.- Create release candidate branch from develop for feature to continue testing

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_2 1e41d67a
$ git push --set-upstream origin rc_2

```

## Commits squashing

Once the basic concepts and use cases are out there, it will go through the way to reduce the amount of commits to manage when merging branches. To squash in Git glossary means to combine multiple commits into one; usually, it is a basic option to consider when a merge is executed, however it will be conducted in several steps when multiple release candidate branches are involved.

### Squashing release candidate to rebase develop (OPTION 1)

**Release candidate** (rc_1) commits could be squashed to simplify the **develop** commits history after rebasing it; nevertheless, it will be conducted through an intermediate branch.

Before to rebase **develop**, new **release candidate** (rc_1-squash) has to be branched off of rc_1; at this point it is needed to undo those commits not existing in develop via reset git command. Once both branches (**develop** and rc_1-squash) have the same commit history, the **release candidate** (rc_1) is merged into new **release candidate** (rc_1-squash) by squash for compacting the whole bugfixing commmits chain into an unique commit.

Finally, **develop** has to be rebased from new **release candidate** (rc_1-squash) to sync both and keep a common seed commits chain; please advise the SHA hashes overwriting for further existing commits on **develop**.

![Squash_1](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_SQUASH_1.png)

Git bash snipped hereunder.

```
1.- Create new relase candidate from current release candidate

$ git checkout rc_1
$ git pull --prune
$ git checkout -b rc_1-squash rc_1
$ git push --set-upstream origin rc_1-squash

2.- Reset commits from rc_1-squash

$ git checkout rc_1-squash
$ git pull --prune
$ git reset --hard c1af9d7e
$ git push --force

3.- Merge rc_1 into rc_1-squash

$ git checkout rc_1
$ git pull --prune
$ git checkout rc_1-squash
$ git pull --prune
$ git merge --squash origin/rc_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'rc_1' into 'rc_1-squash'"
$ git push --force

4.- Rebase develop from release candidate rc_1-squash

$ git checkout rc_1-squash
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/rc_1-squash

... resolve the conflicts ...

$ git push --force

```

### Squashing release candidate to rebase develop (OPTION 2)

Instead of branching off of **release candidate** (rc_1), it could be branched off of **develop** from the same commit which rc_1 was created. This approach avoids to reset commits from new **release candidate** (rc_1-squash).

![Squash_2](https://github.com/dloira/dls_DynamicClassLoading/blob/master/Images/GITFLOW_SQUASH_2.png)

Git bash snipped hereunder.

```
1.- Create new relase candidate from develop

$ git checkout develop
$ git pull --prune
$ git checkout -b rc_1-squash c1af9d7e
$ git push --set-upstream origin rc_1-squash

2.- Merge rc_1 into rc_1-squash

$ git checkout rc_1
$ git pull --prune
$ git checkout rc_1-squash
$ git pull --prune
$ git merge --squash origin/rc_1

... resolve the conflicts ...

$ git add .
$ git commit -am "Merge branch 'rc_1' into 'rc_1-squash'"
$ git push --force

3.- Rebase develop from release candidate rc_1-squash

$ git checkout rc_1-squash
$ git pull --prune
$ git checkout develop
$ git pull --prune
$ git rebase origin/rc_1-squash

... resolve the conflicts ...

$ git push --force

```

## Golden rules

Please consider the following list as the basic reference and use caution in pursuing them; it is really mandatory to check the commit history deeply and backup those branches before to execute whatever command you could not get under control. 

* To merge on **develop** beta features with maturity enough to be backwards-compatible and enabled with feature flag configuration.
* To bugfix on short-term **release candidate** branched off of **develop**. The sooner the feature arrives on Production the best (do not wait for getting the feature 100 ready, releasing not functional small chuncks in hiden mode will help your day by day).
* To rebase **develop** from **release candidate** branches as much as you can; please, let's balance properly the amount and commits relevance (do not wait for fishing the whole bugfixing, rebasing small bunch of commits will help your conflicts resolution).
* To merge on **master** from **release candidate** ready with squash.
* To rebase **develop** from **master** once new commits arrive there.
* To fix Poduction bugs on **hotfix** branched off of **master**.
* To sync **master** commits from hotfix on **develop**, it is mandatory to close all **release candidate** branches alive previously (conduct one by one and in reverse ordering followed to their creation).
* To write merge commit message following the convention: **"Merge branch '**_[PLACEHOLDER branch name FROM]_ **' into '**_[PLACEHOLDER branch name TO]_ **'"** (it will help to find the proper commit in the history)
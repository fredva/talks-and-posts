After using Git for several years, I found myself gradually using more and more advanced Git commands as part of my daily workflow. Soon after I discovered Git rebase, I quickly incorporated it into my daily workflow. Those who are familiar with rebasing know how powerful a tool it is, and how tempting it is to use it all the time. However, I soon discovered that rebasing presents some challenges that are not obvious when you first start doing it. Before presenting them, I'll quickly recap the differences between merging and rebasing.

Let's first consider the basic example where you want to integrate a feature branch with master. By merging, we create a new commit (`g`) that represents the merge between the two branches. The commit graph clearly shows what has happended, and we can see the contours of the "train track" graph familiar from larger Git-repos.

![Example of merging](merge.gif)

Alternatively, we could rebase before merging. The commits are removed and feature is reset to master, after which the commits are re-applied on top of feature. The diffs of these re-applied commits are usually identical to their original counterparts, but they have different parent commits, and hence different SHA-1 keys.

![Example of rebasing](rebase.gif)

We've now changed the base commit of feature from `b` to `c`, literally re-basing it.
Merging feature to master is now a fast-forward merge, because all commits on feature are direct descendants of master.

![Example of fast forward merge](rebase-ff.gif)

Compared to the merge approach, the resulting history is linear with no divergent branches. The improved readability is the reason many people prefer to rebase their branches before merging.

However, this approach comes with a set of challenges that are not that obvious.

Let's say a dependency that's still in use in feature has been removed in master. When rebasing, this will cause the first re-applied commit to break your build, but as long as there are no merge conflicts, the rebase process will continue uninterrupted. The error from the first commit will remain present in all subsequent commits, resulting in a chain of broken commits.

This error is only discovered after the rebase process is finished, and is usually fixed by applying a new bugfix commit (`g`) on top.

![Example of erroneous rebasing](rebase-error.gif)

If you do get conflicts during rebasing however, Git will pause on the conflicting commit, allowing you to fix the conflict before proceeding. Solving conflicts out of context, in the middle of rebasing a long chain of commits, is often confusing, hard to get right, and another source of potential errors.

Introducing errors is extra problematic when it happens during rebasing. This way, new errors are introduced when you rewrite history, and they may disguise genuine bugs that were introduced when history was first written. In particular, this will make it harder to use Git bisect, arguably the most powerful debugging tool in the Git toolbox. As an example, consider the following feature branch. Let's say we introduced a bug towards the end of the branch.

![Example of branch with errors](new-error.png)

You may not discover this bug until weeks after the branch was merged to master. To find the commit that introduced the bug, we might have to search through tens or hundreds of commits. This process can be automated by writing a script that tests for the presence of the bug, and running it automatically through Git bisect, using the command `git bisect run test.sh`.

Bisect will perform a bisection search through the history, identifying the commit that introduced the bug. In the example shown in figure XXX, it succeeds in finding the first buggy commit, since all the broken commits contain the actual bug we are looking for.

![Example of successfull debugging with Git bisect](bisect-success.gif)

On the other hand, if we've introduced additional broken commits during rebasing, bisect will run into trouble. In this case, we hope that Git identifies commit f as the bad one, but it erroneously identifies d instead, since it contains some other error that breaks the test.

![Example of unsuccessfull debugging with Git bisect](bisect-failure.gif)

This problem is greater than it may seem at first.

Why do we use Git at all? Because it is our most important tool for tracking down the source of bugs in our code. Git is our safety net. By rebasing, we give this less priority, in favour of the desire to achieve a linear history.

This is not only a meaningless prioritization. It is bad craftsmanship, and irresponsible towards the rest of our team.

A while back, I had to bisect through several hundred commits to track down a bug in our system. The faulty commit was located in the middle of a long chain of commits that didn't compile, due to a faulty rebase a colleague had performed. This uneccessary and totally avoidable error resulted in me spending nearly a day extra in tracking down the commit.

So how can we avoid these chains of broken commits during rebasing?
One approach could be to let the rebase process finish, test the code to identify any bugs, and go back in history to fix the bugs where they were introduced. To this end, we could use interactive rebasing (LENK DENNE).

Another approach would be to have Git pause during every step of the rebase process, test for any bugs and fix them immediately before proceeding.

This is a cumbersome and error-prone process, and the only reason for doing it would be to achieve a linear history. Does a simpler and better alternative exist?

Yes it does; good old merging solves all our troubles. It's a simple, one-step process, in which we solve all our conflicts at once. The resulting merge commit clearly marks the integration point between our branches, and our history depicts what actually happened, and when it happened.

The importance of keeping your history true should not be underestimated. By rebasing, we are lying to ourselves and to our team. We pretend that the commits were written today, when they were in fact written yesterday, based on another commit. We've taken the commits out of their original context, disguising what actually happened. Can we be sure that the code builds? Can we be sure that the commit messages still make sense? We may believe that we are cleaning up and clarifying our history, but the result may very well be the opposite.

It's impossible to say what errors and troubles the future brings for our codebase. However, you can be sure that it's more likely that the history will be useful in the future if it's true, than if it's untrue.

Although this doesn't feel right we still keep doing it. Why is that? What drives us? What is the point?

I've come to the conclusion that it's about vanity. Rebasing is a purely aestetic move. The apparently clean history appeals to us as developers, but it can't be justified, from neither a technical nor functional standpoint.

![Screenshot from GitUp of Git commit graph](gitup.png)

The chaotic "train track" graphs are intimidating. They certinaly felt that way to me to begin with, but there's no reason to be scared of them. There are a multitude of magnificent tools to analyse and visualise such history, both GUI- and CLI-based. These graphs contain valuable information about what has happened and when it happened, and we gain nothing by linearizing it.

Git is made for, and encourages, non-linear history. Fearing that, you might be better off using a simpler VCS that only supports linear history.

I think you should keep your history true. Get comfortable with tools to analyse it, and don't fall for the temptation to rewrite it. The rewards for rewriting are minimal, but the risks are great. You'll thank me the next time you are bisecting though your history to track down a sneaky bug.

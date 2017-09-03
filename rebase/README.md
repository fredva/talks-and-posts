After using Git for several years, I found myself gradually using more and more advanced Git commands as part of my daily workflow. Soon after I discovered Git rebase, I quickly incorporated it into my daily workflow. Those who are familiar with rebasing knows how powerful a tool it is, and it’s tempting to use it all the time. However, I soon discovered that rebasing presents some risks that are not obvious when you first start using it. Before presenting them, we’ll quickly recap the differences between merging and rebasing.

Let’s first consider the basic example where you want to integrate a feature branch with master. By merging, we create a new commit that represents the merge between the two branches. The commit graph clearly shows what has happended, but we can already see the contours of the train track-graph that we know from larger Git-repos.

![Example of merging](merge.gif)

Alternatively, we could rebase before merging. Our commits are removed and feature is reset to master, after which our commits are re-applied on top of feature. The diffs of these re-applied commits are usually identical to their original counterparts, but they have different parent commits, and hence different SHA-1 keys.

ILL. 2 (rebase gif)

We’ve now changed the base commit of feature from b to c, literally re-basing it.
Merging feature to master is now a fast-forward merge, because all commits on feature are direct descendants of master.

ILL. 4 (ff-merge)

Compared to the merge approach, the resulting history is linear with no divergent branches. The improved readability is the reason many people prefer to rebase their branches before merging.

However, this approach comes with a set of risks that are not that obvious.

Consider figure 5:

ILL. 5 (gif)

Let’s say a dependency has been removed in master. This will cause the first re-applied commit to break your build, but as long as there are no merge conflicts, the rebase process will continue uninterrupted. The error from the first commit will remain present in all subsequent commits, resulting in a chain of broken commits.

We discover this error only after the rebase process is finished, and usually fix it by applying a new bugfix commit (g) on top.

If you do get conflicts during rebasing, git will pause on the conflicting commit, allowing you to fix the conflict before proceeding. Solving conflicts out of context, in the middle of rebasing a long chain of commits, is often confusing, hard to get right, and another source of potential errors.

Introducing errors is extra problematic when it happens during rebasing. This way, new errors are introduced when you rewrite history, and they may disguise genuine bugs that were introduced when history was first written. In particular, this will make it harder to use git bisect, arguably the most powerful debugging tool in the git toolbox. As an example, consider the following example feature branch. Let’s say we introduced a bug towards the end of the branch.

ILL. 6


You may not discover this bug until weeks after the branch was merged to master. To find the commit that introduced the bug, we might have to search through tens or hundreds of commits. This process can be automated by writing a script that tests for the presence of the bug, and running is automatically through git bisect, using the command `git bisect run test.sh`

Bisect will perform a bisection search through the history, identifying the commit that introduced the bug. In the example shown in figure XXX, it succeeds in finding the first buggy commit, since all the broken commits contain the actual bug we are looking for.

ILL. 7 (gif, finner feilen)

On the other hand, if we’ve introduced additional broken commits during rebasing, bisect will run into trouble. In this case, we hope that Git identifies commit f as the bad one, but it erroneously identifies d instead, since it contains some other error that breaks the test.

ILL. 8, finner ikke feilen

This problem is greater than it may seem like on the surface.

Recall why we use Git at all: Git is our safety net, our most important tool for tracking down the source of bugs in our code. By rebasing, we give this goal less priority, in favor of our desire to achieve a linear history.

This is not only a meaningless prioritization. It is bad craftsmanship, and irresponsible towards the rest of our team.

A while back, I had to bisect through several hundred commits to track down a bug in our system. The feaulty commit was located in the middle of a long chain of commits that did‘t compile, due to a feaulty rebase a colleague had performed. This uneccessary and totally avoidable error made me spend nearly a day extra in tracking down the commit.

So how can we avoid these chains of broken commits during rebasing?
One approach could be to let the rebase process finish, test the code to identify any bugs, and go back in history to fix the bugs where they were introduced. To this end, we could use interactive rebasing (LENK DENNE).

Another approach would be to have Git pause during every step of the rebase process, test for any bugs and fix them immediately before proceeding.

This is a cumbersome and error-prone process, and the only reason for doing it would be to achieve a linear history. Does a simpler and better alternative exist?

Yes it does; good old merging solves all our troubles. It’s a simple, one-step process, in which we solve all our conflicts at once. The resulting merge commit clearly marks the integration point between our branches, and our history depicts what actually happened, and when it happened.

The importance of keeping your history true should not be underestimated. By rebasing, we are lying to ourselves and to our team. We pretend that the commits were written today, when they were in fact written yesterday, based on another commit. We’ve taken the commits out of their original context, disguising what actually happened. Can we be sure that the code builds? Can we be sure that the commit messages still make sense? We may believe that we clean up and clarify our history, but the result may very well be the opposite.

It’s impossible to say what errors and troubles the future brings for our codebase. However, you can be sure that it’s more likely that the history will be useful in the future if it’s true, than if it’s untrue.

Many of us have noticed that this doesn’t feel right, but we still keep doing it. Why is that? What drives us? What is the point?

I’ve come to the conclusion that it’s vanity. Rebasing is a purely aestetic move. The apparently clean history appeals to us as developers, but it can’t be justified, not from a technical nor functional standpoint.

ILL. GitUp-graf

The chaotic “train track” graphs are intimidating. They certinaly felt that way to me to begin with, but there’s no reason to be scared of them. There are a multitude of magnificent tools to analyse such history, bot GUI- and CLI-based. These graphs contain valuable information about what has happened and when it happened, and we gain nothing by linearizing it.

Git is made for, and encourages, non-linear history. Fearing that, you might be better off using a simpler VCS that only supports linear history.

I think you should keep your history be true. Get comfortable with tools to analyze, and don’t fall for the temptation to rewrite it. The rewards for rewriting are minimal, but the risks are great. You’ll thank me the next time you are bisecting though your history to track down a sneaky bug.






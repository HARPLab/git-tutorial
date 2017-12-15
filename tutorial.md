How Git/DVCS (distributed version control systems) work
----
2 goals:
- recover lost data/keep a history of changes
- coordinate between lots of different versions (different computers, different branches, different states of development, etc)

Git is an abstraction of the idea of taking lots of backups. We want to record:
- a snapshot of what all files look like
- what the files looked like before that snapshot (equivalently, a reference to the previous node)
- sometimes you might want to name specific commits (tags) or keep a pointer that moves forward as you commit (branches)

Features (different from other VCSs/DVCSs):
- *local* copy of repo
 - each clone has all info and is symmetric (no need to push to a common server, unlike CVS/SVN)
 - push/pull moves changes between clones
 - can push/pull between *any* two clones
 - clone on github is convenient central access point but not actually special
- cryptographically hard to break the tree (commits named by SHA256 hash)
- staging area to decide what workflow changes to add to a commit (unlike Hg)
- ability to rewrite history (this is dangerous, make sure you know what you're doing before doing this; I'm not familiar enough with it to discuss)
- because git stores a copy of each file for each commit (with some caching), it's expensive to add large binary files into git. So make sure you need to before adding your 1.3Gb avi.

Best practices
----
Keep in mind: 
- will this choice make a previous state easy to recover?
- will this choice make it easier for different changes to be merged together?


First, a caveat: These principles were developed by groups of software engineers working together professionally to produce a reliable product used by many people. Some of our academic code (e.g. core robot modules) are similar. Some of our academic code is a repo used by a single person to back up their one-off analysis code. So these principles may vary in application. But you shouldn't break the rules unless you understand them.


So, some general principles:

- Pull before you start working. Save a merge.

- Don't commit any files that you don't want to share with everyone! The `.gitignore` file in the root of the directory will tell git not to track certain files. For example, my `.gitignore` files almost always look like this (without the comments):

```
*.pyc		# don't include the compiled .py files because they might silently cache old versions, let each clone generate them separately. Similarly, if you're in a latex repo, add *.bbl, 
.*project	# your mileage may vary, but I don't commit my Eclipse project file (or the equivalent for your IDE) because all of the paths are custom to my computer.
tmp/*		# if your script generates some output or test logs, you likely don't want to make them part of the permanent record
*~			# if your editor generates lock files (e.g. Emacs), don't commit the lock files
```

In general, avoid doing `git commit -a` unless you're certain everything is worth sharing. It takes an extra few lines to manually add in only the files you care about with `git add my_change.py; git commit`. But if you're trying to track down a bug that was introduced in the code in the few minutes before a demo, you'll be grateful. Similarly, if you try to deploy the code on a new computer, if you e.g. fixed a bug but never ran the code, so you have correct `.py` files but incorrect `.pyc` files, your code will silently still replicate the bug even though if you read the code your fix will be there. Make life easier for yourself.

- Make _atomic_ commits. Basically, each commit should ideally represent a single idea that can be simply summarized in a single commit message. Philosophy varies on this, but a good commit message might be "changed function X to take an additional input." A very bad commit would be one that message is "all changes I made on Dec 18". Again, YMMV with academic code where git is used as backup (I do the second all the time for one-off analysis code), but if you're trying to find the version where function X had only the original commit, being able to (1) read commit messages to figure out which one to go back to and (2) more importantly, actually have exist a single commit to go back to that represents only the change you want rather than having to disentangle a massive collection of unrelated changes will make your life a lot easier. There is lots of discussion of what a good commit should be, and don't worry too much about making them perfect (though we can discuss better/less good practices), but have in mind "if I am trying to revert these changes, or merge them with someone else's, will my future self thank me?".

- Use branches aggressively. There are a number of different workflows for how to use branches, and we can discuss what best fits our different repo styles. But in general, the principle here is *keep different projects as separate as possible*. Many guides suggest that before beginning any self-contained feature, you should create a branch. Then you can do development on that branch, and only once it's complete do you merge it back into the previous tree. Say A and B are working on two different features, Fa and Fb. Let's look at the result for two different philosophies:
 1. A and B work simultaneously on one `develop` branch.
 2. A and B each create separate branches, `feature-Fa` and `feature-Fb`, and work on those branches.

Case 1:
1. A does some partial work and commits to `develop`.
2. Now B does some partial work and commits to `develop`. When B goes to test, it turns out the code is behaving unpredictably because of A's partial work on feature Fa. So B fixes up their code and does some changes to Fa to unbreak the structure and commits it to `develop`.
3. A pulls the code and starts doing work. But somehow B has gone in and messed things up. So A overwrites B's changes and moves on.
4. Now C wants a deployable version of the code to add a minor feature. So C clones `develop`, makes a small change, tests it, and pushes it back. But when C goes to test, everything is broken because of A and B's partial work!

Case 2:
1. A and B create separate branches and do work.
2. When A pushes their first commit, B never sees it because it's on a different branch.
3. When C makes a minor change to the `develop` branch, the partial work from A and B don't cause any problems.
4. After A is done, A merges `feature-Fa` into `develop`. Now when C makes another minor change, they're working from a finished, tested feature and not a partial one.
5. When B goes to merge, they find that there are some conflicts with A! But since A is finished with the feature, B knows what the feature is trying to do (and theoretically has some tests to run to make sure it's not broken), so B can fix the merge with full information.

The standard workflow used in professional settings looks something like this:
- `master` is used only for released versions (e.g. 1.3.2)
- `develop` has ongoing work for the next release version
 - if there are two different types of development going on, e.g. for v1.4 and v2.0, they go on separate branches
- `feature-f1` branches are created off of `develop` for any significant features. when they're merged back in, `develop` is also merged into assorted appropriate branches
- `bugfix-b1` branches are created off of `develop` to fix bugs and are treated like features, though they can be used for a separate release.
- `hotfix-h1` branches are created off of `master` for emergency bugfixes ("hotfixes") and merged back into `master` (with a new version number, v1.3.3) and also into `develop`.
See ["A successful Git branching model"](http://nvie.com/posts/a-successful-git-branching-model/) for detail. Again, it's not clear that the entire formal model is necessary for all of our repos, but the more widely-used a project of ours is (more people use it or more projects use it), the closer we should try to hew to that model.

Takeaways:
----
Make sure everything you do in git makes it easier for your future self to
- go back to a specific version of the code
- not battle with your collaborators.
Everything here is derived from those two principles.




Resources
----
- ["The Git Parable"](http://tom.preston-werner.com/2009/05/19/the-git-parable.html): a story explaining the philosophy of Git
- ["A successful Git branching model"](http://nvie.com/posts/a-successful-git-branching-model/): the standard model used for Git branches throughout professional coding situations
- ["Pro Git"](https://git-scm.com/book/en/v2): good reference
- ["HgInit"](http://hginit.com/01.html): actually a reference on Mercurial (hg), a different DVCS, but a good workthrough of the features and pretty generally applicable.


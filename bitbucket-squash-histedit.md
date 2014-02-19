# Squashing commits using Bitbucket, Mercurial, and the histedit extension

**Bitbucket's Mercurial non-publishing repository feature helps keep your team's history clean. Read this if you miss the ability to interactively rebase before issuing a pull request.**

## Coming from git

One of the common habits git users develop is causal, throwaway commits during feature development.

Git makes it easy to clean up later, and many shops keep their main repo clean by squashing mid-development commits into a single commit upon merge.

	git commit -m 'testing out a feature'
	git commit -m 'checkpoint commit'
	git rebase -i <tree-ish>

Mercurial discourages you from changing anything that enters the `public` [phase](http://mercurial.selenic.com/wiki/Phases#Publishing_Repository).  However, by default pushing code to Bitbucket will change your commits to `public` phase, preventing you from using `histedit` to clean up that code later.

## Non-publishing repository

The solution to this problem is the "non-publishing repository" option in your project's settings.  You can find this by clicking the gear icon on the far left side of the repository navigation. *Ed: Hopefully Atlassian will add this checkbox to the "new repository" form, too.* Once this setting is enabled, commits pushed to your repo will stay in `draft` mode, indicated by a small `D` icon next to the commit id.

## Cleaning up before a pull request

Once you've completed your feature and you're ready for a pull request, squash your commits using the histedit extension.

If you haven't already, add histedit to your `.hgrc` file.

	[plugins]
	histedit =

Then squash the commits.

	hg histedit <commit_id>
	
Your editor will open with a list of commits between `<commit_id>` and `tip` in a format very similar to an interactive rebase in git. Change `pick` to `fold` on all but the first commit to squash all commits into a single commit.
	
	pick fb7b926acbf9 54 Started working on feature
	fold 520a7c7a7265 55 Made some more updates on feature
	fold 4c0ff301d723 56 Fixed a bug!
	
	# Edit history between c2d29f715c6c and 4c0ff301d723
	#
	# Commands:
	#  p, pick = use commit
	#  e, edit = use commit, but stop for amending
	#  f, fold = use commit, but fold into previous commit (combines N and N-1)
	#  d, drop = remove commit from history
	#  m, mess = edit message without changing commit content
	
You'll then be able to edit the final commit message (annoyingly, once per folded commit).  Once you're satisfied, push the commits back up to Bitbucket

	hg push -f

You'll need to force push or you'll get the error `abort: push creates new remote head`.  If you're concerned about having two `default` heads in your Bitbucket repository (we don't worry about this because we create a throwaway fork for every feature branch), you can push with a new branch name `hg push --new-branch-name`.
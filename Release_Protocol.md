# Release Procedure

When it is time to release, use a pull request to put the repository into a releasable state,
allowing testing and edits prior to merging to master.
The following procedure ensures a predictable release.

### 1. Fetch the latest version of the [master branch of the BIDS-specification](https://github.com/bids-standard/bids-specification/tree/master)

You should have a remote, which we will call `upstream`, for the [
bids-standard/bids-specification](https://github.com/bids-standard/bids-specification/) repository:

```Shell
$ git remote get-url upstream
git@github.com:bids-standard/bids-specification.git
```

If you do not, add it with:

```Shell
$ git remote add upstream git@github.com:bids-standard/bids-specification.git
```

Fetch the current repository state and create a new `rel/<version>` branch based on
`upstream/master`.
For example, if releasing version `1.2.0`:

```Shell
$ git fetch upstream
$ git checkout -b rel/1.2.0 upstream/master
```

### 2. Update the version in the changelog and mkdocs.yml

Change the "Unreleased" heading in
[src/CHANGES.md](https://github.com/bids-standard/bids-specification/blob/master/src/CHANGES.md)
to `v<version>`, and link to the target ReadTheDocs URL.
If the target release date is known, include the date in YYYY-MM-DD in parentheses after
the link.

```Diff
- ## Unreleased
+ ## [v1.2.0](https://bids-specification.readthedocs.io/en/v1.2.0/) (2019-03-04)
```

The date can be changed or added later, so accurate prediction is not necessary.

Remove the `-dev` from the version in
[mkdocs.yml](https://github.com/bids-standard/bids-specification/blob/master/mkdocs.yml)
configuration, so the title will be correct for the released specification.
If the version preceding the `-dev` is not the target version, update the version as well.
In the figure below, we update `v1.2.0-dev` to `v1.2.0`.
![dev-to-stable](release_images/site_name_release_1.2dev-1.2.png "dev-to-stable")

### 3. Commit changes and push to upstream

By pushing `rel/` branches to the main repository, the chances of continuous integration
discrepancies is reduced.

```Shell
$ git add src/CHANGES.md mkdocs.yml
$ git commit -m 'REL: v1.2.0`
$ git push -u upstream rel/1.2.0
```

### 4. Open a pull request against the master branch
Important note: The pull request title **must** be named "REL: vX.Y.Z" (*e.g.*, "REL: v1.2.0").

**This will open a period of discussion for 5 business days regarding if we are ready to release.**

Minor revisions may be made using GitHub's [suggestion
feature](https://help.github.com/en/articles/incorporating-feedback-in-your-pull-request).
For larger changes, pull requests should be made against `master`.

**Merging other pull requests during this period requires agreement in this discussion.**

There are no hard-and-fast rules for what other pull requests might be merged, but the focus
should generally be on achieving a self-consistent, backwards-compatible document.
For example, if an inconsistency is noticed, a PR might be necessary to resolve it.
Merging an entire BEP would likely lead to greater uncertainty about self-consistency, and should
probably wait.

If `master` is updated, it should be merged into the `rel/<verison>` branch:

```Shell
$ get fetch upstream
$ git checkout rel/1.2.0
$ git merge upstream/master
$ git push rel/1.2.0
```

### 5. Clean the changelog

Review `src/CHANGES.md` to ensure that the document produces a changelog that is useful to a
reader of the specification.
For example, several small PRs fixing typos might be merged into a single line-item, or less
important changes might be moved down the list to ensure that large changes are more prominent.

### 6. Set release date and merge

On the day of release, the current date should be added to/updated in the changelog in the form
YYYY-MM-DD.
The date should be placed after the link to the versioned URL.
For example:

```Diff
- ## [v1.2.0](https://bids-specification.readthedocs.io/en/v1.2.0/)
+ ## [v1.2.0](https://bids-specification.readthedocs.io/en/v1.2.0/) (2019-03-04)
```

Verify that the pull request title matches "REL: vX.Y.Z" and merge the pull request.

### 7. Tag the release

GitHub's release mechanism does not have all of the features we need, so manually tag the release
in your local repository.
To do this, `fetch` the current state of `upstream` (see step 1), tag `upstream/master`, and
`push` the tag to `upstream`.

```Shell
$ git fetch upstream
$ git tag -a -m "v1.2.0 (2019-03-04)" v1.2.0 upstream/master
$ git push upstream v1.2.0
```

There are four components to the tag command:

1. `-a-` indicates that we want to use an
   [annotated tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging#_creating_tags), which will
   ensure that [`git describe`](https://git-scm.com/docs/git-describe) works nicely with the
   repository.
2. `-m <message>` is the message that will be saved with the tag.
3. `v<version>` is the name of the release and the tag.
4. `upstream/master` instructs `git` to tag the most recent commit on the `master` branch of the
   `upstream` remote.

### 8. Create a GitHub release

Some GitHub processes may only trigger on a GitHub release, rather than a tag push.
To make a GitHub release, go to the [Releases
](https://github.com/bids-standard/bids-specification/releases) page:
![GH-release-1](release_images/GH-release_1.png "GH-release-1")

Click [Draft a new release](https://github.com/bids-standard/bids-specification/releases/new):

![GH-release-2](release_images/GH-release_2.png "GH-release-2")

Set the tag version and release title to "vX.Y.Z", and paste the current changelog as the
description:

![GH-release-3](release_images/GH-release_3.png "GH-release-3")

Click "Publish release".

### 9. Edit the mkdocs.yml file site_name to set a new development version

Please submit a PR with the title `REL: <version>-dev`.
This should be the first merged PR in the new version.
This process is illustrated below.

![stable-to-dev](release_images/site_name_release_1.2-1.3dev.png "stable-to-dev")

Note that the development version number should be larger than the last release, with the
version of the next *intended* release, followed by `-dev`.
For example, after the 1.3.0 release, either `1.3.1-dev` or `1.4.0-dev` would be reasonable, based
on the expected next version.

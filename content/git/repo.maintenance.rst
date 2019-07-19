==================================
Source code Repository maintenance
==================================

:date: 2018-03-23
:modified: 2019-07-19
:tags: git
:category: git
:authors: Malahal Naineni

Source code Repository maintenance
==================================

Since ganesha V2.7 is actively maintained upstream, "ibm2.7" branch
uses rebase to easily identify our patches that are NOT in upstream yet.
It also uses 'merge' strategy to make sure we don't modify the history
and can get back to any old tags that we built in the past. It uses the
strategy outlined here at http://fanf.livejournal.com/128282.html, Use
git-repub.sh script from
https://git.csx.cam.ac.uk/x/ucs/git/git-repub.git while merging.

These are the steps we follow for updating ibm2.7 branch.

#. Make sure you have the latest changes from the ganltc remote as well
   as the upstream repo.  Assuming that "ganltc" remote points to
   https://github.com/ganltc/nfs-ganesha.git and "origin" points to
   https://github.com/nfs-ganesha/nfs-ganesha.git::

    git fetch ganltc && git fetch origin

#. Create the deployment branch "ibm2.7"::
   
    git checkout -b ibm2.7 ganltc/ibm2.7

#. Create a working branch from ibm2.7 from the merge commit's second
   parent::

    git checkout -b working ganltc/ibm2.7^2

#. Perform one of the following steps

    - If we are rebasing on top of the latest V2.7-stable, update the
      checked out working branch by rebasing our patches with the latest
      V2.7-stable. Note that the version patch always fails to merge.
      Fix the version and *modify the version commit message* to reflect
      the new version!::

        git rebase origin/next
        git commit -a --amend # to fix version commit message

    - If we want to add some extra patches, just add them using "git
      cherry-pick" and *create a new version patch and remove the old
      version patch*::

        git cherry-pick -x <commit1>
        git cherry-pick -x <commit2>
        git rebase -i <deep-enough-commit> to reorder and edit the old version
            commit to the top and then modify src/CMakeLists.txt and
            debian/changelog files appropriately with a new version.
        git commit -a --amend to *modify the version commit message*

#. At this point, your working branch has all the patches you need in
   a linear history, no merge commits etc. It should have the version
   commit at the top. This is ready for testing. Run cthon/pynfs with
   this 'working' branch.

#. Use git-repub.sh script to merge (this is due to lack of gits "-s
   theirs" merge strategy option)::

    git-repub.sh --rw working --ff ibm2.7

    The above command may need --force option in case your deployment
    branch ibm2.7 is getting prepared for the first time!

#. Make sure that the deployment branch "ibm2.7" has merge commit as its HEAD::

    gitk ibm2.7 OR "git cat-file -p ibm2.7 | grep ^parent"

#. Make sure that the deployment branch and the working branch have the
   same exact code (no diff output is expected!)::

    git diff ibm2.5 working

#. Checkout the deployment branch "ibm2.7"::

    git checkout ibm2.7
    git tag -s <tag>
    git push ganltc ibm2.7
    git push ganltc <tag>

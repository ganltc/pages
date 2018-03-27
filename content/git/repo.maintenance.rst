==================================
Source code Repository maintenance
==================================

:date: 2018-03-23
:modified: 2018-03-23
:tags: git
:category: git
:authors: Malahal Naineni

Source code Repository maintenance
==================================

Since ganesha V2.5 is very actively maintained upstream, "ibm2.5" branch
uses rebase to easily identify our patches that are NOT in upstream yet.
It also uses 'merge' strategy to make sure we don't modify the history
and can get back to anything that we built in the past. It uses the
strategy outlined here at http://fanf.livejournal.com/128282.html, Use
git-repub.sh script from
https://git.csx.cam.ac.uk/x/ucs/git/git-repub.git while merging.

These are the steps we follow for updating ibm2.5 branch:

#. Make sure you have the latest changes from the ganltc remote as well
   as the upstream repo.  Assuming that "ganltc" remote points to
   https://github.com/ganltc/nfs-ganesha.git and "origin" points to
   https://github.com/nfs-ganesha/nfs-ganesha.git::

    git fetch ganltc && git fetch origin

#. Create a merge branch "ibm2.5"::
   
    git checkout -b ibm2.5 gitlab/ibm2.5

#. Create a working branch from ibm2.5 from the merge commit's second
   parent::

    git checkout -b working ganltc/ibm2.5^2

#. Update the checked out working branch by rebasing our patches with
   the latest upstream. Note that the version patch always fails to
   merge.  Fix the version and modify the commit message to reflect the
   new version! If we want to add our own patches, just add them using
   "git cherry-pick" and create a new version patch and remove the old
   one::

    git rebase origin/next

#. Use git-repub.sh script to merge (this is due to lack of gits "-s
   theirs" strategy option)::

    git-repub.sh --rw working --ff ibm2.5


#. Checkout the new merge branch "ibm2.5". Run regression tests and push
   the new code::

    git checkout ibm2.5
    git push ganltc ibm2.5

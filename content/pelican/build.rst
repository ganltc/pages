===============================
How to build these github pages
===============================

:date: 2017-09-20
:modified: 2017-09-20
:tags: pelican
:category: pelican
:authors: Malahal Naineni
:summary: Steps to build this blog

Software
========

pelican is a blog software written in python that generates html files
from your markdown or restructured text and other markup files. We use
restructured text files here. You need pelican (and ghp-import to make
our life easier).

Install Pelican and ghp-import with
===================================
::

    pip install pelican
    pip install ghp-import

Clone the source of this blog pages
===================================
::

    git clone https://github.com/ganltc/pages.git
    cd pages

    Update or add new restructured text files (.rst extension) in "content"
    directory with any changes you want

Test locally
============
::

    make html    # creates "output" directory from "content"
    Open "output/index.html" with your favourite browser.
    Edit the .rst files until you think they look good in your browser!

Create gh-pages branch with ghp-import tool
===========================================
::

    make publish        # creates "output" directory for publishing
    ghp-import output   # creates gh-pages git branch with "output"

Push gh-pages upstream
======================
::

    git push git@github.com:ganltc/ganltc.github.io gh-pages:master

Finally Push the source (rst files) repo upstream
=================================================
::

    git push origin master

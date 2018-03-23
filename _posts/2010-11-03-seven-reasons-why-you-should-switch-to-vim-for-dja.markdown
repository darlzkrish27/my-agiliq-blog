---
layout: post
title:  "Seven reasons why you should switch to Vim"
date:   2010-11-03 16:16:15+05:30
tags: vim
author: Javed
---
So you want a better IDE for developing django, huh? Why not give good old vim a try?


Use [pathogen](https://github.com/tpope/vim-pathogen) to maintain your vim plugins (and sanity).
Using this, you can clone the repositories listed here to `.vim/bundle/` and start using them
right away.


Also, consider adding your `.vimrc` and `.vim` to a repository. Include `.vimrc` inside `.vim` and
symlink `.vim/.vimrc` to `~/.vimrc` to version control your `.vimrc`.


My vim files can be found [here](https://github.com/tuxcanfly/CalVIM). Also, here's an [imgur album](http://imgur.com/a/MEYkV/1) demonstrating
these plugins in action.


1. Syntax highlighting for django templates
--------------------------------------------
Starting from vim 7.1, syntax highlight for django templates works out
of the box. (Check your vim version using `vim --version` or `:version` within vim)

If you are on an older version, use [this plugin](http://www.vim.org/scripts/script.php?script_id=1487)

2. Django snippets with snipmate
---------------------------------

[SnipMate](https://github.com/msanders/snipmate.vim) provides commonly used "snippets". This eliminates a lot of
boiler plate code genreally required to start a django project from scratch.

Django specific snippets can be found at [robhudson's repository](https://github.com/robhudson/snipmate_for_django)

Its very easy to write custom snippets in case you need them.

3. On-the-fly error checking with pyflakes
-------------------------------------------

This one is not django specific, but it can save a lot of time spent debugging typos or silly mistakes.

[PyFlakes](https://github.com/kevinw/pyflakes-vim) detects errors in your python code and highlights the 
offending lines so you can easily rectify them.

4. Git integration with fugitive
---------------------------------

[Fugitive](https://github.com/tpope/vim-fugitive) is a git plugin for vim.

This one integrates seamlessly with my workflow. You can add/reset files, commit them, view diff, blame, logs all 
within vim!

5. Auto-completion using pysmell
---------------------------------

[Pysmell](https://github.com/orestis/pysmell) is great for autocompleting your django project code.

6. Browse code with taglist
----------------------------

[Taglist](https://github.com/mexpolk/vim-taglist) allows you to view your code structure and jump between
classes/functions.

7. Write faster html using sparkup
-----------------------------------

[Sparkup](https://github.com/bingaman/vim-sparkup) allows you write [zen code](http://code.google.com/p/zen-coding/)

Other tips:
-----------

* set `makeprg` to `python manage.py syncdb`. Run `:make` from your vim to start `syncdb`

* use [nerdtree](https://github.com/scrooloose/nerdtree)/[lusty explorer](https://github.com/vim-scripts/LustyExplorer) for file
management

* use [vimango](https://github.com/tuxcanfly/ViMango) for navigating your django project






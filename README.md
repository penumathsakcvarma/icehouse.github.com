Installation
------------

Install pygments (python-based syntax highlighting) and jekyll. On my mac I just used:

    $ bundle install
    $ sudo easy_install Pygments

With homebrew, instead do:

    $ bundle install
    $ easy_install Pygments
    $ cd /usr/local/bin
    $ ln -s ../Cellar/python/2.7/bin/pygmentize

Usage
-----

    $ jekyll --server --safe --pygments
    $ open http://localhost:4000

Deployment
----------

    $ git push

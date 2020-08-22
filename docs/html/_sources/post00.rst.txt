Day 0: Set up a blog using Sphinx and Github Pages
====================================================

*Date* : **21 Aug 2020**

*Topics* : ``Sphinx``, ``Github pages``

.. highlight:: sh

Let's start with how I setup this blog to be hosted on github pages. Always loved using  `Sphinx <https://www.sphinx-doc.org>`__ and `reStructuredText (or reST in short) <https://docutils.sourceforge.io/rst.html>`__ for documentation. Now let's try it for building a static blog that can be hosted on `Github Pages <https://pages.github.com/>`__.

1. Initiate
--------------- 

#. Create a directory and change into it. (I use the name ``dlm``, which is the abbreviation of my blog name. Replace that with anything you like) : :: 
   
    mkdir -p dlm 
    cd dlm 
   
#. Initiate an empty git repository : :: 
   
    git init 
   
#. Make and activate a python virtual environment : ::

    python3 -m venv env
    source env/bin/activate

#. We do not need this virtual environment to be version-controlled. ::

    echo "env/" > .gitignore

#. Install Sphinx using ``pip`` and collect the list of all dependencies into ``requirements.txt``. ::

    pip install sphinx
    pip freeze > requirements.txt

#. Create a ``README.rst`` at the root and add a short description of what this project is about.      

#. That's almost all for the first step. Let's wind it up with the initial commit. ::

    git add .
    git commit -m "Initial commit."

2. Build source
----------------   

We will separate the source files and generated blog site into two separate branches. Source files will remain in the ``master`` branch while the generated static files will move into a ``docs/`` folder of a new branch, ``gh-pages``. ``Github Pages`` can identify these branch and folder names, so keep the names as they are.

#. Generate the project stub: ::

    sphinx-quickstart

#. Edit the ``Makefile`` and make the following changes in it:

    * Change ``BUILDDIR=build`` into ``BUILDDIR=docs``.

    * Add the following lines to the end of the last section, ``%: Makefile`` : ::

        cp $(BUILDDIR)/html/.nojekyll $(BUILDDIR)/ 
        echo '<meta http-equiv="refresh" content="0; url=./html/index.html" />' > $(BUILDDIR)/index.html    

    * My ``Makefile`` looks as follows after these changes (Changes are highlighted): 
         
    .. image:: images/Makefile.png
  
#. Edit the ``make.bat`` windows script and make changes similiar to above. I am not an expert of windows batch scripts, so leaving it to you.

#. Edit ``source/conf.py`` to make the following changes:

    * Replace relevant uses of words, ``Documentation``, and ``documentation`` with the terms, ``Blog`` and ``blog`` respectively.

    * Add the following lines under the ``Options for HTML Output`` section: ::

        html_title = ""
        html_short_title = project

#. Edit ``source/index.rst`` to make the following changes:

    * Replace relevant uses of words, ``Documentation``, and ``documentation`` with the terms, ``Blog`` and ``blog`` respectively.

    * Add the following line at the end of section named, ``toctree``: ::

        :reversed:

3. Generate and separate blog site
-----------------------------------   

#. It's time to generate the blog site from our source: ::

    make html

   This will generate the site into ``docs/html`` and an redirecting index file will be created at ``docs/index.html`` too. The empty file, ``.nojekyll`` prevents github pages from rendering the site as a Jekyll-based site.

#. Limit ``master`` branch for source files only.

    * Copy/move the ``docs/`` content to somewhere outside the repository: ::

        mv docs ../latest_docs
      
    * Exclude ``docs/`` from version control:

        As said earlier, our ``master`` branch will continue to track source files only. So let's direct git to ignore changes in docs. For this, edit ``.gitignore`` and add a line, ``docs/``. Now our ``.gitignore`` file in the master branch will look like, ::

            env/
            docs/

    * Add and commit source files: ::
                     
        git add .
        git commit -m "First commit of source."

#. Bring back generated site to a new branch.        

    * Create a new git branch named, ``gh-pages``: ::

        git checkout -b gh-pages

    * Now we are in ``gh-pages`` branch. Here, we do not want to track any source files. So edit ``.gitignore`` again to look like : ::

        env/
        source/
        Makefile
        make.bat

    * Copy back the latest contents of ``docs/``: ::

        mv ../latest_docs ./docs         

    * Add and Commit changes in ``gh-pages`` branch:  ::

        git add .
        git commit -m "First commit of generated blog."

4. Push to Github and get hosted
-----------------------------------   

#. Go to Github, login and create a new repository, having the same name as our local repo. You do not need to add README, LICENSE etc to it.

#. Move back to the master branch, if not already in it: ::

    git checkout master

#. Add the URL of that remote github repo into our local repo's config: ::

    git remote add origin WRITE_YOUR_REPO_URL_HERE 

#. Push both branches to github: ::

    git push --all origin

#. Go back to github and visit the new repo's page. Go to its ``Settings`` tab, scroll down to find the ``Github Pages`` section, choose branch ``gh-pages`` and folder ``docs`` as source and click the ``Save`` button.

#. Now you will be greeted with a "Your site is published" alert. Go to the address specified in it and you can see the site up and running.


5. Add new posts
-------------------

#. Move back to the master branch, if not already in it: ::

    git checkout master

#. Write your post in ``reStructuredText`` format and place it in ``source`` directory of ``master`` branch.

#. Add the filename of your post (without the ``.rst`` extension) to the end of ``toctree`` section of ``source/index.rst``.

#. Run ``make html`` and check the output by opening the ``docs/index.html`` file in a browser.

#. Edit your post source and repeat the above step until you get a satisfactory output.

#. Copy/move the ``docs`` folder to somewhere outside this repository.

#. Add and commit all changes in the ``master`` branch.

#. Checkout ``gh-pages`` branch, copy or move back the latest ``docs`` from the previous step, add and commit changes.

#. Push both branches to github by running ``git push --all origin`` command.

#. This updates your site over github pages and publishes the new post.


# Tutorial: Basic tutorial for building a Conda package

Continuum's Conda toolset provides cross-platform binary package management.
Originally created to distribute Python packages, conda is in active
development and can now install packages in [R](https://binstar.org/r/r-plyr),
[Javascript](https://binstar.org/wakari/nodejs), [Lua](https://binstar.org/alexbw/luajit), [Perl](https://binstar.org/dan_blanchard/perl) and more.
This tutorial will teach you what goes into building conda packages. 

Creating a conda package from a typical Python package built with `python setup.py install` is a one-line command. 
Other times, when dependencies are more complex or the packages files were placed
in an ad-hoc way relative to one another, conda requires a bit more nudging. We'll start with the easy cases and work through to the hard ones.


## Install conda and conda-build

#### Make sure you have Anaconda or Miniconda

If you do scientific computing, you've probably already installed Continuum Analytics' [Anaconda](https://store.continuum.io/cshop/anaconda/) distribution, which includes conda as its' package manager. If not, download [Miniconda](http://conda.pydata.org/miniconda.html). 

##### Linux and OSX

On Linux and OSX, download miniconda from http://conda.pydata.org/miniconda.html

    $ chmod +x Miniconda-latest-Linux-x86_64.sh  # or Miniconda-latest-MacOSX-x86_64.sh
    $ bash Miniconda-latest-Linux-x86_64.sh # or Miniconda-latest-MacOSX-x86_64.sh

(use the filename for the installer you downloaded)

##### Windows

On Windows, open and run the .exe installer.

You can hit q to go to the end of the license.
The script asks where to install conda. **select the default location**. It asks whether or not to add the path to your environment. **say yes**.

You will not need administrative access to run Conda or manage Conda
environments.

Once Conda is installed, relaunch a terminal window and confirm that:

    $ conda info

finds the executable where you have just installed it.  If you did add Anaconda
or Miniconda to your PATH (the default option) you will have to supply its
path explicitly.

It's a useful habit to do:

    $ conda update conda

with a fresh install. 

#### Install conda-build

You now can install conda packages. To make them yourself, you need conda-build.

    $ conda install conda-build

On Linux, you may also need to install ``patchelf``.

    $ conda install patchelf
    
##  Conda packages come from Conda Recipes
    
    A conda package comes from a conda recipe, which is 
    <Explain conda recipes>


## Elementary Conda Package Building


#### Using conda skeleton to build from a PyPI package


It is easy to build a skeleton recipe for any Python package that is hosted on
`PyPI
<https://pypi.python.org/>`_.


Let's generate a new conda recipe for `pyinstrument <https://github.com/joerick/pyinstrument>`_, by using
`PyPI <https://pypi.python.org/>`_ metadata:

.. code-block:: bash

    $ cd ~/
    $ conda skeleton pypi pyinstrument

You should verify the
existence of the ``meta.yaml``, ``build.sh``, and ``bld.bat`` files in a newly created
directory called ``pyinstrument``.

You should always check the ``meta.yaml`` file output from the ``skeleton``
subcommand invocation, as it is not perfect, and it often requires some things
to be filled in manually. For instance, some packages do not specify
dependencies properly in their setup.py, so they will need to be added
manually. Some hints for Python package dependencies:

* If you get an error saying that setuptools downloading is disabled during
  conda build, this means that setuptools is trying to download and install a
  dependency of the package. Dependencies should be split out into separate
  packages, so this is disallowed, as it would create a single package with
  all the dependencies. The fix is to add this package as both a run and build
  time dependency in the ``requirements`` section of the meta.yaml.

* If the build or test fails with an ImportError for an external library, it
  means it needs to depend on it.

* If a build fails with an ImportError for pkg_resources, it means it needs to
  depend on setuptools (or alternately, you can write a patch for the package
  that removes the runtime dependence on pkg_resources).

Now, it should be straightforward to use the ``conda build`` tool. Let's try it:

.. code-block:: bash

    $ conda build pyinstrument

Now everything works great and the package was saved to
~/miniconda/conda-bld/linux-64/pyinstrument-0.12_py270.tar.bz2 file. The exact
location of the file may be a little different for you, depending on where you
have conda installed and what operating system you are using. conda build will
tell you where the file is located at the end of the build.

Later you will upload this package to Binstar, but for now, you can install it
with the ``--use-local`` flag.

.. code-block:: bash

   $ conda install --use-local pyinstrument
   
### Clone conda recipe examples from GitHub

The `conda recipes <https://github.com/conda/conda-recipes>`_ repo on GitHub
has many example conda recipes. This is not a necessary step to build your own
packages, but it's a very useful resource to investigate existing recipes for
similar packages to the one you are trying to build. In many cases, a recipe
for the package you are trying to build may already exist there. If you do not
have git installed you will need to install it first.

    $ git clone https://github.com/conda/conda-recipes

After getting familiar with full process of package building, feel free to add
your own new recipes to this repository by making a pull request.


#### Writing the meta.yaml by hand


Suppose you stick with the same package, ``pyinstrument``, but don't start
from conda skeleton pypi. You can fill in the values in ``meta.yaml``
manually, based on other conda recipes and information about where to download
the tarball.

The easiest way to do this is to start from an existing example from the
`conda-recipes <https://github.com/conda/conda-recipes>`_ repo.  Take the
``meta.yaml`` file from the ``pyfaker`` package:

.. code-block:: yaml

    package:
      name: pyfaker

    source:
      git_tag: 0.3.2
      git_url: https://github.com/tpn/faker.git

    requirements:
      build:
        - python
        - setuptools

      run:
        - python

    test:
      imports:
        - faker

    about:
      home: http://www.joke2k.net/faker
      license: MIT

With a search on the [GitHub site of
pyinstrument](https://github.com/joerick/pyinstrument) and some sensible
choices for substitutions, you get a makeshift .yaml for ``pyinstrument``:

.. code-block:: yaml

    package:
      name: pyinstrument

    source:
      git_tag: 0.12
      git_url: https://github.com/joerick/pyinstrument.git

    requirements:
      build:
        - python
        - setuptools

      run:
        - python

    test:
      imports:
        - pyinstrument

    about:
      home: https://github.com/joerick/pyinstrument
      license: BSD
      summary: "Call stack profiler for Python. Inspired by Apple's Instruments.app"

This seems reasonable. Being sure to supply ``build.sh`` and ``bld.bat`` files in the
same directory. For Python packages, these can just be ``python setup.py
install`` for both.

Note that the original recipe was built using a tarball from PyPI:

.. code-block:: yaml

    fn: pyinstrument-0.12.tar.gz
    url: https://pypi.python.org/packages/source/p/pyinstrument/pyinstrument-0.12.tar.gz

whereas this one was built using a git url and a git tag:

.. code-block:: yaml

      git_tag: 0.12
      git_url: https://github.com/joerick/pyinstrument.git

Both ways should work just fine. As the source should be identical. For some C
packages, building from a tarball may be preferable to building from git, as
building from git requires more build tools, such as autoconf. For pure Python
packages such as pyinstrument, there is generally no difference.

There is more information about all the values that can go in the
``meta.yaml`` file on the :ref:`build` page.

#### Uploading packages to `binstar.org <https://binstar.org>`__


All of above steps produce one object - the package (a tar.bz2
archive). During package building process you were asked if the package should
be uploaded to `binstar.org <https://binstar.org>`__. To get more info about
`binstar.org <https://binstar.org>`__ visit `the Binstar documentation page
<http://docs.binstar.org/>`_.

Here is a minimal summary. First, you need the ``binstar`` command line
client. Install this tool by running:

.. code-block:: bash

   $ conda install binstar

Now you should `register an account on binstar.org
<https://binstar.org/account/register>`_.  Then login with the ``binstar``
command

.. code-block:: bash

   $ binstar login

One this is done, you are ready to upload your package.

.. code-block:: bash

    $ binstar upload ~/miniconda/conda-bld/linux-64/pyinstrument-0.12-py27_0.tar.bz

Replace this path with the path to the package printed at the end of conda
build.

If you always want conda build to upload to Binstar after a successful build,
you can run

.. code-block:: bash

   $ conda config --set binstar_upload yes

If you then want to install these packages, it is recommended to add your
Binstar channel to the conda configuration, so that conda will always search
your channel in addition to the default Continuum ones.

.. code-block:: bash

   $ conda config --add channels your_username

(replace ``your_username`` with your Binstar username).

#### Searching for already existing packages

You have two methods to accomplish this task. First option is to use ``conda
search``. ``conda`` searches all the channels configured from the ``.condarc``
file for the given string. You can see what channels are searched by running

.. code-block:: bash

   $ conda info

If there is no ``.condarc`` file, conda only searches the default Continuum
channels, which are officially maintained by `Continuum Analytics
<http://continuum.io/>`_. This includes all the packages from the Anaconda
distribution.

For example, to search for the ``sympy`` package, type

.. code-block:: bash

    $ conda search sympy

Sometimes you may want to follow a person who is constantly building new
packages and publishing them on `binstar.org <https://binstar.org>`__. To be
able to use those packages you have to add appropriate channel of that person
to your ``~/.condarc`` file, just like this:

.. code-block:: yaml

    channels:
        - defaults
        - asmeurer
        - mutirri

In this example you have added two new channels (of ``asmeurer`` and
``mutirri``).  Note that for Binstar channels, it is only necessary to enter
the username of the person. You can also add the full channel url, like
``https://conda.binstar.org/asmeurer``.

From now on you will be able to search for any package in these users' package
lists, and install them too.

Another way to do this is through the command line using the ``conda config``
option.

.. code-block:: yaml

   $ conda config --add channels asmeurer
   $ conda config --add channels mutirri

The order of the channels matters. If two channels have the same version of
the same package, the one from higher in the list will be chosen.  The ``conda
config`` command will always prepend the channel (add it to the top of the
list).

You can also search all of Binstar, without adding channels to the
``.condarc`` file using the ``binstar`` command.

.. code-block:: bash

    $ binstar search sympy

This command will search through all users' packages on `binstar.org
<http://binstar.org>`__.  **But remember**, to be able to install a package
which was found in this way, you still have to add the appropriate user's channel
to your ``.condarc`` file.

Another way to do this is to run the conda tool with the ``-c`` flag, which
adds the channel just for that one command. For example, to install the
``pyinstrument`` package from ``asmeurer``'s Binstar channel, run

.. code-block:: bash

    $ conda install -c asmeurer pyinstrument

For more information about this topic, see the `binstar.org documentation page
<http://docs.binstar.org/>`_.

#### Additional References


`Using PyPI packages for conda <http://www.peterbronez.com/Using%20PyPi%20Packages%20with%20Conda>`_

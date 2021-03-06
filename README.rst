===============================
esss_fix_format
===============================


.. image:: https://img.shields.io/travis/ESSS/esss_fix_format/master.svg
        :target: https://travis-ci.org/esss/esss_fix_format


Simple code formatter and pre-commit checker used internally by ESSS.

* Imports sorted using `isort <https://pypi.python.org/pypi/isort>`_;
* Trim right spaces;
* Expand tabs;


Install
-------

``esss_fix_format`` is installed automatically by the ``ben10`` environment file.

If you want to install it manually, execute from your project's conda environment:

.. code-block:: sh

    conda install esss_fix_format

If executed from the root environment (or another environment) isort could classify wrongly some modules.

To obtain coloring support, you may also install ``colorama``:    

.. code-block:: sh

    conda install colorama
    
(``colorama`` is not installed by default because we are still assessing if this is safe).    


Usage
-----

Use ``fix-format`` (or ``ff`` for short) to reorder imports and format source code automatically.

1. To format files and/or directories::

    fix-format <file1> <dir1> ...


2. Format only modified files in Git::

    fix-format --commit

   Or more succinctly::

    ff -c


Migrating a project to use fix-format
-------------------------------------

Follow this steps to re format an entire project and start using the pre-commit hook:

1. You should have ``ff`` available in your environment already:

    .. code-block:: sh

        $ ff --help
        Usage: ff-script.py [OPTIONS] [FILES_OR_DIRECTORIES]...

          Fixes and checks formatting according to ESSS standards.

        Options:
          -k, --check   check if files are correctly formatted
          --stdin       read filenames from stdin (1 per line)
          -c, --commit  use modified files from git
          --help        Show this message and exit.


2. Search for all usages of ``LoadCppModule`` function (from ``coilib50``), and for each file that
   uses it add ``isort:skipfile`` to the docstring:

    .. code-block:: python

        """
        isort:skip_file
        """

   Commit using ``-n`` to skip the current hook.

3. If there are any sensitive imports in your code which you wouldn't like to ``ff`` to touch, use
   a comment to prevent ``isort`` from touching it:

    .. code-block:: python

        ConfigurePyroSettings()  # must be called before importing Pyro4
        import Pyro4  # isort:skip

4. Execute:

    .. code-block:: sh

        $ cd /path/to/repo/root
        $ ff .

   After it completes, make sure there are no problems with the files:

    .. code-block:: sh

        $ ff . --check

   .. note::
        if the check fails, try running it again; there's a rare
        `bug in isort <https://github.com/timothycrosley/isort/issues/460>`_ that might
        require to run ``ff /path/to/repo/root`` twice.

   Commit:

    .. code-block:: sh

        $ git commit -anm "Apply fix-format on all files" --author="Dev <dev@esss.com.br>"


5. Execute ``codegen`` and check if no files were modified:

    .. code-block:: sh

        $ inv codegen

6. Push and run your branch on CI.

7. If all goes well, finally make ``codegen`` install the hook automatically in your ``tasks.py``:

    .. code-block:: python

        @ctask
        def _codegen(ctx, cache='none', flags=''):
            ns.tasks['constants'](ctx)
            ns.tasks['hooks'](ctx)


8. Profit!


Developing (conda)
------------------

Create a conda environent (using Python 3 here) and install it in development mode:

.. code-block:: sh

    $ conda create -n esss-fix-format-py3 python=3
    $ source activate esss-fix-format-py3
    $ pip install -e .
    $ pip install . -r requirements_dev.txt
    $ pytest

When implementing changes, please do it in a separate branch and open a PR.

Licensed under the MIT license.

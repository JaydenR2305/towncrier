Hear ye, hear ye, says the ``towncrier``
========================================

.. image:: https://img.shields.io/github/workflow/status/twisted/towncrier/CI/master
    :alt: GitHub Actions
    :target: https://github.com/twisted/towncrier/actions?query=branch%3Amaster

.. image:: https://img.shields.io/codecov/c/github/twisted/towncrier/master
    :alt: Codecov
    :target: https://app.codecov.io/gh/twisted/towncrier/branch/master


``towncrier`` is a utility to produce useful, summarised news files for your project.
Rather than reading the Git history as some newer tools to produce it, or having one single file which developers all write to, ``towncrier`` reads "news fragments" which contain information `useful to end users`.

Used by `Twisted <https://github.com/twisted/twisted>`_, `pytest <https://github.com/pytest-dev/pytest/>`_, `pip <https://github.com/pypa/pip/>`_, `BuildBot <https://github.com/buildbot/buildbot>`_, and `attrs <https://github.com/python-attrs/attrs>`_, among others.


Philosophy
----------

``towncrier`` delivers the news which is convenient to those that hear it, not those that write it.

That is, by duplicating what has changed from the "developer log" (which may contain complex information about the original issue, how it was fixed, who authored the fix, and who reviewed the fix) into a "news fragment" (a small file containing just enough information to be useful to end users), ``towncrier`` can produce a digest of the changes which is valuable to those who may wish to use the software.
These fragments are also commonly called "topfiles" or "newsfiles" in Twisted parlance.

``towncrier`` works best in a development system where all merges involve closing a ticket.


Quick Start
-----------

Install from PyPI::

    python3 -m pip install towncrier

.. note::

   ``towncrier``, as a command line tool, works on Python 3.7+ only.
   It is usable by projects written in other languages, provided you specify the project version either in the configuration file or on the command line.
   For Python-compatible projects, the version can be discovered automatically.

In your project root, add a ``towncrier.toml`` or a ``pyproject.toml`` file (if both files exist, the first will take precedence).
You can configure your project in two ways.
To configure it via an explicit directory, add:

.. code-block:: toml

    [tool.towncrier]
    directory = "changes"

Alternatively, to configure it relative to a (Python) package directory, add:

.. code-block:: toml

    [tool.towncrier]
    package = "mypackage"
    package_dir = "src"
    filename = "NEWS.rst"

.. note::

    ``towncrier`` will also look in ``pyproject.toml`` for configuration if ``towncrier.toml`` is not found.

For the latter, news fragments (see "News Fragments" below) should be in a ``newsfragments`` directory under your package.
Using the above example, your news fragments would be ``src/myproject/newsfragments/``).

.. tip::

    To prevent git from removing the ``newsfragments`` directory, make a ``.gitignore`` file in it with::

        !.gitignore

    This will keep the folder around, but otherwise "empty".

``towncrier`` needs to know what version your project is, and there are three ways you can give it:

- For Python-compatible projects, a ``__version__`` in the top level package.
  This can be either a string literal, a tuple, or an `Incremental <https://github.com/hawkowl/incremental>`_ version.

- Manually passing ``--version=<myversionhere>`` when interacting with ``towncrier``.

- Definining a ``version`` option in a configuration file:

.. code-block:: ini

    [tool.towncrier]
    # ...
    version = "1.2.3"  # project version if maintained separately

To create a new news fragment, use the ``towncrier create`` command.
For example::

    towncrier create 123.feature

To produce a draft of the news file, run::

    towncrier build --draft

To produce the news file for real, run::

    towncrier build

This command will remove the news files (with ``git rm``) and append the built news to the filename specified by the ``filename`` configuration option, and then stage the news file changes (with ``git add``).
It leaves committing the changes up to the user.

If you wish to have content at the top of the news file (for example, to say where you can find the tickets), put your text above a rST comment that says::

  .. towncrier release notes start

``towncrier`` will then put the version notes after this comment, and leave your existing content that was above it where it is.


News Fragments
--------------

``towncrier`` has a few standard types of news fragments, signified by the file extension.
These are:

- ``.feature``: Signifying a new feature.
- ``.bugfix``: Signifying a bug fix.
- ``.doc``: Signifying a documentation improvement.
- ``.removal``: Signifying a deprecation or removal of public API.
- ``.misc``: A ticket has been closed, but it is not of interest to users.

The start of the filename is the ticket number, and the content is what will end up in the news file.
For example, if ticket #850 is about adding a new widget, the filename would be ``myproject/newsfragments/850.feature`` and the content would be ``myproject.widget has been added``.


Further Options
---------------

Towncrier has the following global options, which can be specified in the toml file:

.. code-block:: toml

    [tool.towncrier]
    package = ""
    package_dir = "."
    single_file = true  # if false, filename is formatted like `title_format`.
    filename = "NEWS.rst"
    directory = "directory/of/news/fragments"
    version = "1.2.3"  # project version if maintained separately
    name = "arbitrary project name"
    template = "path/to/template.rst"
    start_string = "Text used to detect where to add the generated content in the middle of a file. Generated content added after this text. Newline auto added."
    title_format = "{name} {version} ({project_date})"  # or false if template includes title
    issue_format = "format string for {issue} (issue is the first part of fragment name)"
    underlines = "=-~"
    wrap = false  # Wrap text to 79 characters
    all_bullets = true  # make all fragments bullet points

If ``single_file`` is set to ``true`` or unspecified, all changes will be written to a single
fixed newsfile, whose name is literally fixed as the ``filename`` option. In each run of ``towncrier build``,
content of new changes will append at the top of old content, and after ``start_string`` if the
``start_string`` already appears in the newsfile. If the corresponding ``top_line``, which is formatted
as the option 'title_format', already exists in newsfile, ``ValueError`` will be raised to remind
you "already produced newsfiles for this version".

If ``single_file`` is set to ``false`` instead, each versioned ``towncrier build`` will generate a
separate newsfile, whose name is formatted as the patten given by option ``filename``.
For example, if ``filename="{version}-notes.rst"``, then the release note with version "7.8.9" will
be written to the file "7.8.9-notes.rst". If the newsfile already exists, its content
will be overwriten with new release note, without throwing a ``ValueError`` warning.

If ``title_format`` is unspecified or an empty string, the default format will be used.
If set to ``false``, no title will be created.
This can be useful if the specified template creates the title itself.

Furthermore, you can customize each of your own fragment types using:

.. code-block:: toml

    [tool.towncrier]
	# To add custom fragment types, with default setting, just add an empty section.
    [tool.towncrier.feat]
    [tool.towncrier.fix]

	# Custom fragment types can have custom attributes
	# that are used when rendering the result based on the template.
    [tool.towncrier.chore]
        name = "Other Tasks"
        showcontent = false



Automatic pull request checks
-----------------------------

To check if a feature branch adds at least one news fragment, run::

    towncrier check

By default this compares the current branch against ``origin/master``. You can use ``--compare-with`` if the trunk is named differently::

    towncrier check --compare-with origin/main

The check is automatically skipped when the main news file is modified inside the branch as this signals a release branch that is expected to not have news fragments.

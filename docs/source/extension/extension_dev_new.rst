.. Copyright (c) Jupyter Development Team.
.. Distributed under the terms of the Modified BSD License.

.. _developer_extensions:

Adding New Features to JupyterLab with Extensions
=================================================

JupyterLab extensions offer you the power to add virtually any new feature
or customization you can imagine to JupyterLab:

- Invent new features that you can share with the world
- Tailor workflows and behaviors to suit the needs
of your team or project
- Change the app's look and style
- ...and more!

Extensions are an important part of JupyterLab's design, meant to encourage
reusability, extensibility, and swappability of core parts of the program.
The JupyterLab app itself is composed of a core application object and
a set of extensions. [TODO add more links for context, architecture, etc]

JupyterLab extensions provide nearly every function in JupyterLab, including
notebooks, document editors and viewers, code consoles, terminals, themes,
the file browser, contextual help system, debugger, and settings editor.
Extensions even provide more fundamental parts of the application, such
as the menu system, status bar, and the underlying communication mechanism
with the [server](ADD LINK OR FOOTNOTE HERE).

You can read a more in-depth explanation of extensions below, after the
quick links section.

.. note::
    Your extensions may break with new releases of JupyterLab. As noted in :ref:`versioning_notes`,
    JupyterLab development and release cycles follow semantic versioning, so we recommend planning
    your development process to account for possible future breaking changes that may disrupt users
    of your extensions. Consider documenting your maintenance plans to users in your project, or
    setting an upper bound on the version of JupyterLab your extension is compatible with in your
    project's package metadata.

Basic information about extensions is split across these two articles:

- Extension Development (this page)
  - Gives high level info about extension development and workflows
  - For late-stage extension development and release, explains how
    to package and (optionally) publish your extension to a package
    repository so you can share your extensions publicly
- [JupyterLab Plugins](LINK)
  - A detailed, low level guide to plugins, the fundamental pieces of
    extension functionality
  - Explains how to write extensions with the plugin API, its design
    and usage, and recommended software design schemes

Quick Links
===========

.. note::
    Skip to the next section if you want to continue reading a more detailed
    explanation of extensions

Here are some links to other helpful articles that may assist you on your extension
development journey.

**Tutorials**

- :ref:`extension_tutorial`: A tutorial to learn how to make a simple JupyterLab extension.
- :ref:`Making Extensions Compatible with Multiple Applications Tutorial <multiple_ui_extensions>`
  A tutorial for making extensions that work in both JupyterLab, Jupyter Notebook 7+ and more
- The `JupyterLab Extension Examples Repository <https://github.com/jupyterlab/extension-examples>`_: A short tutorial series to learn how to develop extensions for JupyterLab by example.
- :ref:`developer-extension-points`: A list of the most common JupyterLab extension points.
- Another common pattern for extending JupyterLab document widgets with application plugins is covered in :ref:`documents`.

How Extensions Work
===================

A JupyterLab extension is made up of a number of JupyterLab plugins bundled
together into a package [ADD_LINK], so plugins form the basic building blocks
of functionality in your extensions: They will be extremely important while
you are writing your extension, and they are essential to understand if you
want to use the JupyterLab tools and API effectively.

.. note::
    The [plugins page](LINK) contains CRITICAL information about extension development,
    so make sure you read it when you're getting started with extensions

While building your extension, you will be using standard plugins (provided
to you by the JupyterLab API) to access parts of the JupyterLab application
itself, and to add functionality to your own extension.

You will also be authoring some of your own plugins (as many as you like),
and they can (optionally) be made available for reuse by other extensions
by following some of the recommended plugin design schemes (read more about
those, like the "Provider-Consumer Pattern" at the plugin page [ADD_LINK]).

Extensions are typically built using a few common tools:

- [The JupyterLab API](LINK): A programming toolkit that hooks into various
  features of the app to enable customization. Here you will find a reference
  for the available parts of the JupyterLab application that you can interact
  with inside your plugins.

- The [Lumino](LINK) toolkit: Provides some user interface components and
  other basic features like intra-app communication between different
  parts of the program (Widget and Signal[ADD_LINK] are common examples).

- An extension template: A template is a collection of pre-prepared files
  that you can adapt to make the process of building your extension easier.
  These templates work with [templating software](ADD_LINK) that makes
  downloading and updating the template when there are changes more
  convenient. See the Appendix [ADD_LINK] for a list of templates.

Both Lumino and the JupyterLab API are written in TypeScript [LINK], which
is typically used for making JupyterLab extensions.

.. _dependency-injection-basic-info:

.. _services:

Plugins: The Basic Building Blocks
----------------------------------

.. note::
    Main article: [JupyterLab Plugins](ADD_LINK).
    
    The [plugins page](LINK) contains CRITICAL information about extension development,
    so make sure you read it when you're getting started with extensions

One of the foundational features of the [JupyterLab plugin system](ADD_LINK) is that
plugins can interact with other plugins. Here's how it works:

- A plugin can provide a "service object" (or just "service") to JupyterLab,
  which makes it available for use by other plugins
- A "service" can be any JavaScript value, and typically is a JavaScript
  object with methods and data attributes
- Your plugins can request service objects provided by other plugins

For example, the built-in plugin that supplies the JupyterLab main menu
provides a :ref:`mainmenu` service object to the system with a method for
adding a new top-level menu, and attributes to interact with existing
top-level application menus.

This design is called the ["Provider-Consumer Pattern"](ADD_LINK) in
JupyterLab, and it is a type of [dependency injection pattern](ADD_LINK).

.. note::
    Read more about the general concept of software "design patterns" here.

For a detailed breakdown of the different types of available plugins, and XX,
see the plugin page [ADD_LINK].

The Structure of an Extension
-----------------------------

A look at the stuff inside an extension template, package.json etc.

Extension Development Workflows
-------------------------------

DIAGRAM ABOUT BUILD PROCESS ETC

EXPLAIN NPM, PACKAGE REPOSITORIES ETC




.. _testing_with_jest:

Testing your extension
----------------------

.. note::

    We highly recommend using the `extension template <https://github.com/jupyterlab/extension-template>`_ to set up tests configuration.

There are a number of helper functions in ``testutils`` in this repo (which
is a public ``npm`` package called ``@jupyterlab/testutils``) that can be used when
writing tests for an extension.  See ``tests/test-application`` for an example
of the infrastructure needed to run tests.

If you are using `jest <https://jestjs.io/>`__ to test your extension, you will
need to transpile the jupyterlab packages to ``commonjs`` as they are using ES6 modules
that ``node`` does not support.

To transpile jupyterlab packages, you need to install the following package:

.. code-block:: shell

   jlpm add --dev jest @types/jest ts-jest @babel/core@^7 @babel/preset-env@^7

Then in ``jest.config.js``, you will specify to use babel for js files and ignore
all node modules except the ES6 modules:

.. code-block:: javascript

    const jestJupyterLab = require('@jupyterlab/testutils/lib/jest-config');

    const esModules = ['@jupyterlab/'].join('|');

    const baseConfig = jestJupyterLab(__dirname);

    module.exports = {
      ...baseConfig,
      automock: false,
      collectCoverageFrom: [
        'src/**/*.{ts,tsx}',
        '!src/**/*.d.ts',
        '!src/**/.ipynb_checkpoints/*'
      ],
      coverageReporters: ['lcov', 'text'],
      testRegex: 'src/.*/.*.spec.ts[x]?$',
      transformIgnorePatterns: [
        ...baseConfig.transformIgnorePatterns,
        `/node_modules/(?!${esModules}).+`
      ]
    };

Finally, you will need to configure babel with a ``babel.config.js`` file containing:

.. code-block:: javascript

   module.exports = require('@jupyterlab/testutils/lib/babel.config');















Preparing Your Extension for Release
------------------------------------

.. note::
    For tips about how to structure your extension and plugins, see the
    plugins document. This section describes late stage steps for packaging
    and (optionally) publishing your finished extension.

When your extension is almost ready for release, FOOBAR








.. _distributing_prebuilt_extensions:

Distributing a prebuilt extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prebuilt extensions can be distributed by any system that can copy the prebuilt assets into an appropriate location where JupyterLab can find them. The `official extension template <https://github.com/jupyterlab/extension-template>`_ shows how to distribute prebuilt extensions via Python pip or conda packages. A system package manager, or even just an administrative script that copies directories, could be used as well.

To distribute a prebuilt extension, copy its :ref:`output directory <outputDir>` to a location where JupyterLab will find it, typically  ``<sys-prefix>/share/jupyter/labextensions/<package-name>``, where ``<package-name>`` is the JavaScript package name in the ``package.json``. For example, if your JavaScript package name is ``@my-org/my-package``, then the appropriate directory would be ``<sys-prefix>/share/jupyter/labextensions/@my-org/my-package``.

The JupyterLab server makes the ``static/`` files available via a ``/labextensions/`` server handler. The settings and themes handlers in the server also load settings and themes from the prebuilt extension directories. If a prebuilt extension has the same name as a source extension, the prebuilt extension is preferred.

.. _install.json:

Packaging Information
"""""""""""""""""""""

Since prebuilt extensions are distributed in many ways (Python pip packages, conda packages, and potentially in many other packaging systems), the prebuilt extension directory can include an extra file, ``install.json``, that helps the user know how a prebuilt extension was installed and how to uninstall it. This file should be copied by the packaging system distributing the prebuilt extension into the top-level directory, for example ``<sys-prefix>/share/jupyter/labextensions/<package-name>/install.json``.

This ``install.json`` file is used by JupyterLab to help a user know how to manage the extension. For example, ``jupyter labextension list`` includes information from this file, and ``jupyter labextension uninstall`` can print helpful uninstall instructions. Here is an example ``install.json`` file::

   {
     "packageManager": "python",
     "packageName": "mypackage",
     "uninstallInstructions": "Use your Python package manager (pip, conda, etc.) to uninstall the package mypackage"
   }

* ``packageManager``: This is the package manager that was used to install the prebuilt extension, for example, ``python``, ``pip``, ``conda``, ``debian``, ``system administrator``, etc.
* ``packageName``: This is the package name of the prebuilt extension in the package manager above, which may be different than the package name in ``package.json``.
* ``uninstallInstructions``: This is a short block of text giving the user instructions for uninstalling the prebuilt extension. For example, it might instruct them to use a system package manager or talk to a system administrator.

.. _dev_trove_classifiers:

PyPI Trove Classifiers
""""""""""""""""""""""

Extensions distributed as Python packages may declare additional metadata in the form of
`trove classifiers <https://pypi.org/classifiers>`__. These improve the browsing
experience for users on `PyPI <https://pypi.org/search>`__. While including the license,
development status, Python versions supported, and other topic classifiers are useful
for many audiences, the following classifiers are specific to Jupyter and JupyterLab.

.. code-block::

    Framework :: Jupyter
    Framework :: Jupyter :: JupyterLab
    Framework :: Jupyter :: JupyterLab :: 1
    Framework :: Jupyter :: JupyterLab :: 2
    Framework :: Jupyter :: JupyterLab :: 3
    Framework :: Jupyter :: JupyterLab :: 4
    Framework :: Jupyter :: JupyterLab :: Extensions
    Framework :: Jupyter :: JupyterLab :: Extensions :: Mime Renderers
    Framework :: Jupyter :: JupyterLab :: Extensions :: Prebuilt
    Framework :: Jupyter :: JupyterLab :: Extensions :: Themes

Include each relevant classifier (and its parents) to help describe what your package
provides to prospective users in your ``setup.py``, ``setup.cfg``, or ``pyproject.toml``.
In particular ``Framework :: Jupyter :: JupyterLab :: Extensions :: Prebuilt`` is used by
the extension manager to get the available extensions from PyPI.org.

.. hint::

    For example, a theme, only compatible with JupyterLab 3, and distributed as
    a ready-to-run, prebuilt extension might look like:

    .. code-block:: python

        # setup.py
        setup(
            # the rest of the package's metadata
            # ...
            classifiers=[
                "Framework :: Jupyter",
                "Framework :: Jupyter :: JupyterLab",
                "Framework :: Jupyter :: JupyterLab :: 3",
                "Framework :: Jupyter :: JupyterLab :: Extensions",
                "Framework :: Jupyter :: JupyterLab :: Extensions :: Prebuilt",
                "Framework :: Jupyter :: JupyterLab :: Extensions :: Themes",
            ]
        )

    This would be discoverable from, for example, a
    `PyPI search for theme extensions <https://pypi.org/search/?c=Framework+%3A%3A+Jupyter+%3A%3A+JupyterLab+%3A%3A+Extensions+%3A%3A+Themes>`__.

.. _source_dev_workflow:



.. _ext-author-companion-packages:

Companion packages
^^^^^^^^^^^^^^^^^^

If your extension depends on the presence of one or more packages in the
kernel, or on a notebook server extension, you can add metadata to indicate
this to the extension manager by adding metadata to your package.json file.
The full options available are::

    "jupyterlab": {
      "discovery": {
        "kernel": [
          {
            "kernel_spec": {
              "language": "<regexp for matching kernel language>",
              "display_name": "<regexp for matching kernel display name>"   // optional
            },
            "base": {
              "name": "<the name of the kernel package>"
            },
            "overrides": {   // optional
              "<manager name, e.g. 'pip'>": {
                "name": "<name of kernel package on pip, if it differs from base name>"
              }
            },
            "managers": [   // list of package managers that have your kernel package
                "pip",
                "conda"
            ]
          }
        ],
        "server": {
          "base": {
            "name": "<the name of the server extension package>"
          },
          "overrides": {   // optional
            "<manager name, e.g. 'pip'>": {
              "name": "<name of server extension package on pip, if it differs from base name>"
            }
          },
          "managers": [   // list of package managers that have your server extension package
              "pip",
              "conda"
          ]
        }
      }
    }


A typical setup for e.g. a jupyter-widget based package will then be::

    "keywords": [
        "jupyterlab-extension",
        "jupyter",
        "widgets",
        "jupyterlab"
    ],
    "jupyterlab": {
      "extension": true,
      "discovery": {
        "kernel": [
          {
            "kernel_spec": {
              "language": "^python",
            },
            "base": {
              "name": "myipywidgetspackage"
            },
            "managers": [
                "pip",
                "conda"
            ]
          }
        ]
      }
    }


Currently supported package managers are ``pip`` and ``conda``.




.. _disabledExtensions:

Disabling other extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``jupyterlab.disabledExtensions`` field gives a list of extensions or plugins to disable when this extension is installed, with the same semantics as the ``disabledExtensions`` field of :ref:`page_config.json <page_configjson>`. This is useful if your extension overrides built-in extensions. For example, if an extension replaces the ``@jupyterlab/filebrowser-extension:share-file`` plugin to :ref:`override the "Copy Shareable Link" <copy_shareable_link>` functionality in the file browser, it can automatically disable the ``@jupyterlab/filebrowser-extension:share-file`` plugin with::

        "jupyterlab": {
          "disabledExtensions": ["@jupyterlab/filebrowser-extension:share-file"]
        }

To disable all plugins in an extension, give the extension package name, e.g., ``"@jupyterlab/filebrowser-extension"`` in the above example.










Distributing Extensions
-----------------------

WHAT IS A PACKAGE?? EXPLAIKN

Extensions can be distributed in two ways:

- A *prebuilt extension* (since JupyterLab 3.0) distributes a bundle of JavaScript code prebuilt from a source extension that can be loaded into JupyterLab without rebuilding JupyterLab. In this case, the extension developer uses tools provided by JupyterLab to compile a source extension into a JavaScript bundle that includes the non-JupyterLab JavaScript dependencies, then distributes the resulting bundle in, for example, a Python pip or conda package. Installing a prebuilt extensions does not require Node.js.
- [DEPRECATED] A *source extension* is a JavaScript (npm) package that exports one or more plugins. Installing a source extension requires a user to rebuild JupyterLab. This rebuilding step requires Node.js and may take a lot of time and memory, so some users may not be able to install a source extension. However, the total size of the JupyterLab code delivered to a user's browser may be reduced compared to using prebuilt extensions. See :ref:`deduplication` for the technical reasons for rebuilding JupyterLab when a source extension is installed.

An extension can be published both as a source extension on NPM and as a prebuilt extension (e.g., published as a Python package). In some cases, system administrators may even choose to install a prebuilt extension by directly copying the prebuilt bundle to an appropriate directory, circumventing the need to create a Python package. If a source extension and a prebuilt extension with the same name are installed in JupyterLab, the prebuilt extension takes precedence.

Because prebuilt extensions do not require a JupyterLab rebuild, they have a distinct advantage in multi-user systems where JupyterLab is installed at the system level. On such systems, only the system administrator has permissions to rebuild JupyterLab and install source extensions. Since prebuilt extensions can be installed at the per-user level, the per-environment level, or the system level, each user can have their own separate set of prebuilt extensions that are loaded dynamically in their browser on top of the system-wide JupyterLab.

.. tip::

   We recommend publishing prebuilt extensions in Python packages for user convenience.












.. _deduplication:

Deduplicating Dependencies
^^^^^^^^^^^^^^^^^^^^^^^^^^

One important concern and challenge in the JupyterLab extension system is deduplicating dependencies of extensions instead of having extensions use their own bundled copies of dependencies. For example, the Lumino widgets system on which JupyterLab relies for communication across the application requires all packages use the same copy of the ``@lumino/widgets`` package. :ref:`Tokens <tokens>` identifying plugin services also need to be shared across the providers and consumers of the services, so dependencies that export tokens need to be deduplicated.

The ``jupyterlab.sharedPackages`` field controls how dependencies are bundled, shared, and deduplicated with prebuilt extensions.

JupyterLab automatically deduplicates the entire dependency tree between source extensions when it rebuilds itself during a source extension installation. Deduplication between source and prebuilt extensions, or between prebuilt extensions themselves, is a more nuanced problem (for those curious about implementation details, this deduplication in JupyterLab is powered by the Webpack 5.0 `module federation system <https://webpack.js.org/concepts/module-federation/>`__). JupyterLab comes with a reasonable default strategy for deduplicating dependencies for prebuilt extensions. The ``jupyterlab.sharedPackages`` object in an extension's ``package.json`` enables an extension author to modify the default deduplication strategy for a given dependency with three boolean options. The keys of this object are dependency package names, and the values are either ``false`` (signifying that dependency should not be shared/deduplicated), or objects with up to three fields:

* ``bundled``: if ``true`` (default), the dependency is bundled with the extension and is made available as one of the copies available to JupyterLab. If ``false``, the dependency is not bundled with the extension, so the extension will use a version of the dependency from a different extension.
* ``singleton``: if ``true``, the extension will always prefer to use the copy of the dependency that other extensions are using, rather than using the highest version available. The default is ``false``.
* ``strictVersion``: if ``true``, the extension will always make sure the copy of the dependency it is using satisfies the dependency version range it requires.

By default, JupyterLab deduplicates direct dependencies of prebuilt extensions with direct dependencies of other source and prebuilt extensions, choosing the highest version of a dependency available to JupyterLab. JupyterLab chooses reasonable default options when using tokens and services from core JupyterLab packages. We suggest the following ``sharedPackages`` configurations when using tokens provided by packages other than core JupyterLab packages (see :ref:`services` for more details about using tokens).

.. _dedup_provide_service:

Deduplicating (Providers)
"""""""""""""""""""""""""

When an extension (the "provider") is providing a service identified by a token that is imported from a dependency ``token-package``, the provider should configure the dependency as a singleton. This makes sure the provider is identifying the service with the same token that others are importing. If ``token-package`` is not a core package, it will be bundled with the provider and available for consumers to import if they :ref:`require the service <dedup_require_service>`.

.. code-block:: json

   "jupyterlab": {
     "sharedPackages": {
       "token-package": {
         "singleton": true
        }
      }
    }

.. _dedup_require_service:

Deduplicating (Required Services)
"""""""""""""""""""""""""""""""""

When an extension (the "consumer") is requiring a service provided by another extension (the "provider"), identified by a token imported from a package (the ``token-package``, which may be the same as the provider), the consumer should configure the dependency ``token-package`` to be a singleton to ensure the consumer is getting the exact same token the provider is using to identify the service. Also, since the provider is providing a copy of ``token-package``, the consumer can exclude it from its bundle.

.. code-block:: json

   "jupyterlab": {
     "sharedPackages": {
       "token-package": {
         "bundled": false,
         "singleton": true
        }
      }
    }

.. _dedup_optional_service:

Deduplicating (Optional Services)
"""""""""""""""""""""""""""""""""

When an extension (the "consumer") is optionally using a service identified by a token imported from a package (the ``token-package``), there is no guarantee that a provider is going to be available and bundling ``token-package``. In this case, the consumer should only configure ``token-package`` to be a singleton:

.. code-block:: json

   "jupyterlab": {
     "sharedPackages": {
       "token-package": {
         "singleton": true
        }
      }
    }

.. TODO: fill out the following text to a more complete explanation of how the deduplication works.

   Prebuilt extensions need to deduplicate many of their dependencies with other prebuilt extensions and with source extensions. This deduplication happens in two phases:

   1. When JupyterLab is initialized in the browser, the core Jupyterlab build (including all source extensions) and each prebuilt extension can share copies of dependencies with a package cache in the browser.
   2. A source or prebuilt extension can import a dependency from the cache while JupyterLab is running.

   The main options controlling how things work in this deduplication are as follows. If a package is listed in this sharing config, it will be requested from the package cache.

   * ``bundled`` - if true, a copy of this package is also provided to the package cache. If false, we will request a version from the package cache. Set this to false if we know that the package cache will have the package and you do not want to bundle a copy (perhaps to make your prebuilt bundle smaller).
   ``singleton`` - if true, makes sure to use the same copy of a dependency that others are using, even if it is not the right version.
   ``strictVersion`` - if true, throw an error if we would be using the wrong version of a dependency.

































Appendix
========

**Extension templates**

JupyterLab provides already-prepared template projects that you can adapt
while building your extension to make the process quicker and easier. These
templates work with [templating software](ADD_LINK) that makes downloading
and updating the template when there are changes more convenient.

We provide several templates to create JupyterLab extensions:

- `extension-template <https://github.com/jupyterlab/extension-template>`_: Create a JupyterLab extension using `copier <https://copier.readthedocs.io/>`_
- [DEPRECATED] `extension-cookiecutter-ts <https://github.com/jupyterlab/extension-cookiecutter-ts>`_: Create a JupyterLab extension using `cookiecutter <https://cookiecutter.readthedocs.io>`_ (use the copier template instead).

**API Reference Documentation**

Here is some autogenerated API documentation [ADD_FOOTNOTE] for JupyterLab and Lumino packages:

- `JupyterLab API Documentation <../api/>`_
- `Lumino API Documentation <https://lumino.readthedocs.io/en/latest/api/index.html>`_







In the following discussion, the plugin that is providing a service to the
system is the *provider* plugin, and the plugin that is requiring and using
the service is the *consumer* plugin. Note that these kinds of *provider*
and *consumer* plugins are fundamental parts of JupyterLab's Provider-Consumer
pattern (which is a type of `dependency-injection <https://en.wikipedia.org/wiki/Dependency_injection>`_
pattern).









######################
MISC FEATURES I DUNNO


Extension CSS
^^^^^^^^^^^^^

If your extension has a top-level ``style`` key in ``package.json``, the CSS file it points to will be included on the page automatically.

A convention in JupyterLab for deduplicating CSS on the page is that if your extension has a top-level ``styleModule`` key in ``package.json`` giving a JavaScript module that can be imported, it will be imported (as a JavaScript module) instead of importing the ``style`` key CSS file as a CSS file.
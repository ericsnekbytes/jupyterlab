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

While building your extension, you will be using standard plugins (provided
to you by the JupyterLab API) to access parts of the JupyterLab application
itself, and to add functionality to your own extension.

You will also be authoring some of your own plugins (as many as you like),
and they can (optionally) be made available for reuse by other extensions
by following some of the recommended plugin design schemes (read more about
those, like the "Provider-Consumer Pattern" at the plugin page [ADD_LINK]).

Extensions are typically built using a few common tools:

- The JupyterLab API [LINK]: A programming toolkit that hooks into various
  features of the app to enable customization. Here you will find a reference
  for the available parts of the JupyterLab application that you can interact
  with inside your plugins.

- The Lumino [LINK] toolkit: Provides some user interface components and
  other basic features like intra-app communication between different
  parts of the program (Widget and Signal[ADD_LINK] are common examples).

- An extension template: A template is a collection of pre-prepared files
  that you can adapt to make the process of building your extension easier.
  These templates work with [templating software](ADD_LINK) that makes
  downloading and updating the template when there are changes more
  convenient. See the Appendix [ADD_LINK] for a list of templates.

Both Lumino and the JupyterLab API are written in TypeScript [LINK], which
is the recommended language for making JupyterLab extensions.

.. _dependency-injection-basic-info:

.. _services:

Plugins: The Basic Building Blocks
----------------------------------

.. note::
    Main article: The JupyterLab Plugin System [ADD_LINK]

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












.. _schemaDir:

Plugin Settings
^^^^^^^^^^^^^^^

JupyterLab exposes a plugin settings system that can be used to provide
default setting values and user overrides. A plugin's settings are specified with a JSON schema file. The ``jupyterlab.schemaDir`` field in ``package.json`` gives the relative location of the directory containing plugin settings schema files.

The setting system relies on plugin ids following the convention ``<source_package_name>:<plugin_name>``. The settings schema file for the plugin ``plugin_name`` is ``<schemaDir>/<plugin_name>.json``.

For example, the JupyterLab ``filebrowser-extension`` package exports the ``@jupyterlab/filebrowser-extension:browser`` plugin. In the ``package.json`` for ``@jupyterlab/filebrowser-extension``, we have::

        "jupyterlab": {
          "schemaDir": "schema",
        }

The file browser setting schema file (which specifies some default keyboard shortcuts and other settings for the filebrowser) is located in ``schema/browser.json`` (see `here <https://github.com/jupyterlab/jupyterlab/blob/main/packages/filebrowser-extension/schema/browser.json>`__).

See the
`fileeditor-extension <https://github.com/jupyterlab/jupyterlab/tree/main/packages/fileeditor-extension>`__
for another example of an extension that uses settings.

Please ensure that the schema files are included in the ``files`` metadata in ``package.json``.

When declaring dependencies on JupyterLab packages, use the ``^`` operator before a package version so that the build system installs the newest patch or minor version for a given major version. For example, ``^4.0.0`` will install version 4.0.0, 4.0.1, 4.1.0, etc.

A system administrator or user can override default values provided in a plugin's settings schema file with the :ref:`overrides.json <overridesjson>` file.

.. _disabledExtensions:

Disabling other extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``jupyterlab.disabledExtensions`` field gives a list of extensions or plugins to disable when this extension is installed, with the same semantics as the ``disabledExtensions`` field of :ref:`page_config.json <page_configjson>`. This is useful if your extension overrides built-in extensions. For example, if an extension replaces the ``@jupyterlab/filebrowser-extension:share-file`` plugin to :ref:`override the "Copy Shareable Link" <copy_shareable_link>` functionality in the file browser, it can automatically disable the ``@jupyterlab/filebrowser-extension:share-file`` plugin with::

        "jupyterlab": {
          "disabledExtensions": ["@jupyterlab/filebrowser-extension:share-file"]
        }

To disable all plugins in an extension, give the extension package name, e.g., ``"@jupyterlab/filebrowser-extension"`` in the above example.

.. _deduplication:

Deduplication of Dependencies
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``jupyterlab.sharedPackages`` field controls how dependencies are bundled, shared, and deduplicated with prebuilt extensions.

One important concern and challenge in the JupyterLab extension system is deduplicating dependencies of extensions instead of having extensions use their own bundled copies of dependencies. For example, the Lumino widgets system on which JupyterLab relies for communication across the application requires all packages use the same copy of the ``@lumino/widgets`` package. :ref:`Tokens <tokens>` identifying plugin services also need to be shared across the providers and consumers of the services, so dependencies that export tokens need to be deduplicated.

JupyterLab automatically deduplicates the entire dependency tree between source extensions when it rebuilds itself during a source extension installation. Deduplication between source and prebuilt extensions, or between prebuilt extensions themselves, is a more nuanced problem (for those curious about implementation details, this deduplication in JupyterLab is powered by the Webpack 5.0 `module federation system <https://webpack.js.org/concepts/module-federation/>`__). JupyterLab comes with a reasonable default strategy for deduplicating dependencies for prebuilt extensions. The ``jupyterlab.sharedPackages`` object in an extension's ``package.json`` enables an extension author to modify the default deduplication strategy for a given dependency with three boolean options. The keys of this object are dependency package names, and the values are either ``false`` (signifying that dependency should not be shared/deduplicated), or objects with up to three fields:

* ``bundled``: if ``true`` (default), the dependency is bundled with the extension and is made available as one of the copies available to JupyterLab. If ``false``, the dependency is not bundled with the extension, so the extension will use a version of the dependency from a different extension.
* ``singleton``: if ``true``, the extension will always prefer to use the copy of the dependency that other extensions are using, rather than using the highest version available. The default is ``false``.
* ``strictVersion``: if ``true``, the extension will always make sure the copy of the dependency it is using satisfies the dependency version range it requires.

By default, JupyterLab deduplicates direct dependencies of prebuilt extensions with direct dependencies of other source and prebuilt extensions, choosing the highest version of a dependency available to JupyterLab. JupyterLab chooses reasonable default options when using tokens and services from core JupyterLab packages. We suggest the following ``sharedPackages`` configurations when using tokens provided by packages other than core JupyterLab packages (see :ref:`services` for more details about using tokens).

.. _dedup_provide_service:






















.. _application_object:

Application Object
""""""""""""""""""

A Jupyter front-end application object is given to a plugin's ``activate`` function as its first argument. The application object has a number of properties and methods for interacting with the application, including:

-  ``commands`` - an extensible registry used to add and execute commands in the application.
-  ``docRegistry`` - an extensible registry containing the document types that the application is able to read and render.
-  ``restored`` - a promise that is resolved when the application has finished loading.
-  ``serviceManager`` - low-level manager for talking to the Jupyter REST API.
-  ``shell`` - a generic Jupyter front-end shell instance, which holds the user interface for the application. See :ref:`shell` for more details.

See the JupyterLab API reference documentation for the ``JupyterFrontEnd`` class for more details.















More Information
================

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

Extension CSS
^^^^^^^^^^^^^

If your extension has a top-level ``style`` key in ``package.json``, the CSS file it points to will be included on the page automatically.

A convention in JupyterLab for deduplicating CSS on the page is that if your extension has a top-level ``styleModule`` key in ``package.json`` giving a JavaScript module that can be imported, it will be imported (as a JavaScript module) instead of importing the ``style`` key CSS file as a CSS file.
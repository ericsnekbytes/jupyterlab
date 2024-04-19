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

Learn how to write JupyterLab extensions with these guides:

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

Extensions can be distributed in two ways:

- A *prebuilt extension* (since JupyterLab 3.0) distributes a bundle of JavaScript code prebuilt from a source extension that can be loaded into JupyterLab without rebuilding JupyterLab. In this case, the extension developer uses tools provided by JupyterLab to compile a source extension into a JavaScript bundle that includes the non-JupyterLab JavaScript dependencies, then distributes the resulting bundle in, for example, a Python pip or conda package. Installing a prebuilt extensions does not require Node.js.
- [DEPRECATED] A *source extension* is a JavaScript (npm) package that exports one or more plugins. Installing a source extension requires a user to rebuild JupyterLab. This rebuilding step requires Node.js and may take a lot of time and memory, so some users may not be able to install a source extension. However, the total size of the JupyterLab code delivered to a user's browser may be reduced compared to using prebuilt extensions. See :ref:`deduplication` for the technical reasons for rebuilding JupyterLab when a source extension is installed.

An extension can be published both as a source extension on NPM and as a prebuilt extension (e.g., published as a Python package). In some cases, system administrators may even choose to install a prebuilt extension by directly copying the prebuilt bundle to an appropriate directory, circumventing the need to create a Python package. If a source extension and a prebuilt extension with the same name are installed in JupyterLab, the prebuilt extension takes precedence.

Because prebuilt extensions do not require a JupyterLab rebuild, they have a distinct advantage in multi-user systems where JupyterLab is installed at the system level. On such systems, only the system administrator has permissions to rebuild JupyterLab and install source extensions. Since prebuilt extensions can be installed at the per-user level, the per-environment level, or the system level, each user can have their own separate set of prebuilt extensions that are loaded dynamically in their browser on top of the system-wide JupyterLab.

.. tip::

   We recommend publishing prebuilt extensions in Python packages for user convenience.












Types of Plugins
^^^^^^^^^^^^^^^^

JupyterLab supports several types of plugins (some with extras restrictions and limitations):

-  **Application plugins:** Application plugins are the fundamental building block of JupyterLab functionality. Application plugins interact with JupyterLab and other plugins by requiring services provided by other plugins, and optionally providing their own service to the system. Application plugins in core JupyterLab include the main menu system, the file browser, and the notebook, console, and file editor components.
-  **Mime renderer plugins:** Mime renderer plugins are simplified, restricted ways to extend JupyterLab to render custom mime data in notebooks and files. These plugins are automatically converted to equivalent application plugins by JupyterLab when they are loaded. Examples of mime renderer plugins that come in core JupyterLab are the pdf viewer, the JSON viewer, and the Vega viewer.
-  **Theme plugins:** Theme plugins provide a way to customize the appearance of JupyterLab by changing themeable values (i.e., CSS variable values) and providing additional fonts and graphics to JupyterLab. JupyterLab comes with light and dark theme plugins.



In the following discussion, the plugin that is providing a service to the
system is the *provider* plugin, and the plugin that is requiring and using
the service is the *consumer* plugin. Note that these kinds of *provider*
and *consumer* plugins are fundamental parts of JupyterLab's Provider-Consumer
pattern (which is a type of `dependency-injection <https://en.wikipedia.org/wiki/Dependency_injection>`_
pattern).









By splitting up your features into multiple plugins, they can be reused by
other extensions, or swapped out so that users can easily modify small pieces
of your extension.



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


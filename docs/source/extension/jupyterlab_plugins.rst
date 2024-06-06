.. Copyright (c) Jupyter Development Team.
.. Distributed under the terms of the Modified BSD License.

JupyterLab Plugins
==================

.. note::
    If you haven't read the main extension document, you should start there
    first. Extensions give you the power to build new features for JupyterLab,
    this document explains the underlying pieces that make up an extension.

Plugin objects are the basic building blocks of functionality in the larger [JupyterLab extension system](ADD_LINK).
Because each JupyterLab extension is made up of a number of bundled plugins,
understanding how to work with them, and how to author your own plugins, is
essential for using the JupyterLab tools and API effectively. While building
your extension, you will be using plugins to do some of these things:

- Access built-in parts of the JupyterLab application itself
  - See Common Extensions Points [ADD_LINK] for a list of existing plugins you can use
- Add existing or built-in functionality to your own extension
- Author new functionality with custom plugins you design
- Substitute built-in parts of JupyterLab or other extensions with
  plugins of your own design

Your plugins can also (optionally) be made available for reuse by other
extensions by following some of the recommended plugin design schemes
desccribed here (or referenced in other parts of this document).

By splitting up your extension's features into multiple plugins, you can take
advantage of the same kind of flexibility and versatility that powers the
JupyterLab application: Your plugins can be reused by other extensions, or
swapped out so that users (or other teams inside your company, for example)
can easily modify small pieces of your extension.

At the highest level, a JupyterLab plugin is a Javascript object with
several metadata fields. Let's take a quick look at an example below (this
is an "application plugin", you will learn more about those later):

.. code-block:: typescript

   const plugin: JupyterFrontEndPlugin<MyToken> = {
     id: 'my-extension:plugin',
     description: 'Provides a new service.',
     autoStart: true,
     requires: [ILabShell, ITranslator],
     optional: [ICommandPalette],
     provides: MyToken,
     activate: activateFunction
   };

When JupyterLab loads your extension, it will read information from these
fields, and will then run the ``activate`` function you supplied in your
``activate`` field. You will use the ``activate`` function to perform the
necessary setup steps to launch your extension.

A more detailed explanation of these fields is included later in this
document, along with details about the different kinds of plugins and what
they are used for. For now, it's helpful to summarize by saying that
these fields drive the flexibility of the extension system.

Plugins Interacting with Each Other
-----------------------------------

One of the foundational features of the JupyterLab plugin system is that
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

JupyterLab's Provider-Consumer Pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

That design scheme, where some plugins "provide" service objects, and others
"consume" them, is called the "Provider-Consumer Pattern" in JupyterLab, which
is a type of "dependency injection" pattern in software design [ADD_LINK].

.. note::
    Read more about the general concept of software "design patterns" here

The Provider-Consumer Pattern is what powers the reusability, extensibility,
and swappability of core parts of the JupyterLab application, and of the
extension system as a whole. By (optionally) following this design scheme,
you can achieve the same kind of flexibility in your own extensions.

To make requests for service objects that you can use in your code, or to
provide your own service objects that others can use, you will list them in
the metadata fields on your plugin objects.

The details are described below.

How Plugin Metadata Powers Reusability in JupyterLab
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each plugin uses some properties (the ``requires`` and ``optional`` properties)
to request features it wants which are provided by other plugins that have been
loaded into JupyterLab. When your plugin requests features, the system sends
them to your plugin's activate function if they're available.

See the example plugin below, which is a "consumer" of the "IStatusBar":

.. code::

  const plugin: JupyterFrontEndPlugin<void> = {
    id: 'shout_button_message:plugin',
    description: 'An extension that adds a button to the right toolbar',
    autoStart: true,
    optional: [IStatusBar],
    activate: (app: JupyterFrontEnd, statusBar: IStatusBar | null) => {
      console.log('JupyterLab extension shout_button_message is activated!');

      // Create a ShoutWidget and add it to the interface in the right sidebar
      const shoutWidget: ShoutWidget = new ShoutWidget(statusBar);
      shoutWidget.id = 'JupyterShoutWidget';  // Widgets need an id
      app.shell.add(shoutWidget, 'right');
    }
  };

Here, you can see the ``optional`` property, which is a list of optional
services this plugin wants (with just a single item, IStatusBar). You
can also see the ``activate`` property of the plugin, which is a callable
(function) that the plugin system will call for you when your plugin is
loaded.

About the "activate" function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It's important to note that the arguments to your ``activate`` callable will
depend on what things you request in your ``optional`` and ``requires`` plugin
properties, so remember to add arguments for any service objects you request
into your activate function's arguments.

When JupyterLab calls your plugin's ``activate`` function, it will always
pass an application as the first argument, then it will pass any ``required``
objects (in the order you specify them), then any ``optional``objects (again,
in the order you specify them).

By returning an object from your activate function, you become a ``provider``
in JupyterLab's provider-consumer pattern (ADD_LINK), and other plugins can use ("consume")
this object (the "service object") in their extensions. Read more about this
in the ``Making Your Plugin a Provider`` (ADD_LINK) section below.

.. _application_object:

The Application Object
""""""""""""""""""""""

JupyterLab will always pass a Jupyter front-end application object to a plugin's
``activate`` function as its first argument. You can use the application object to
add your widget(s) to the JupyterLab interface (for instance) and it has a number
of other properties and methods you can use to interact with documents, data and
other features of the program.

-  ``commands`` - an extensible registry used to add and execute commands in the application.
-  ``docRegistry`` - an extensible registry containing the document types that the application is able to read and render.
-  ``restored`` - a promise that is resolved when the application has finished loading.
-  ``serviceManager`` - low-level manager for talking to the Jupyter REST API.
-  ``shell`` - a generic Jupyter front-end shell instance, which holds the user interface for the application. See :ref:`shell` for more details.

See the JupyterLab API reference documentation for the ``JupyterFrontEnd`` class for more details.

How Requesting Features Works
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you designate a feature in the ``requires`` list of your plugin, JupyterLab
will only load your plugin if that feature is available (it will fail to load
otherwise).

By designating a feature in the ``optional`` list, JupyterLab will
pass you an object for it (if it's available) or ``null`` if it's not.

Both of these behaviors can be used to enable compatibility with multiple
Jupyter applications (like JupyterLab + Jupyter Notebook 7), which you can
read more about in the :ref:`Compatibility Guide <extension_multiple_ui>`.

When JupyterLab starts up, it checks which plugins need which services, and
carefully arranges the order in which plugins are activated to ensure that
a provider (ADD_LINK) of a service is activated before its consumers.

In order to identify a particular service object/feature, JupyterLab uses
"Token" objects (instances of the Lumino Token class [ADD_LINK]), so the items
that you list in your ``requires`` and ``optional`` fields are not actually
the service objects or the plugins themselves, they are actually token objects
that are used to identify and fetch the actual service objects they are
associated with.

**Not everyone needs to use tokens extensively!**

If you're planning on splitting up your extension into multiple plugins and
making those available for reuse by others in JupyterLab's extension system,
it's good to know a little more about tokens (and how to make your own).

If not, you can just think of them as identifiers.

A consumer might list a token as ``optional`` when the service it identifies
is not critical to the consumer, but would be nice to have if the service is
available. For example, a consumer might list the status bar service as
optional so that it can add an indicator to the status bar if it is available,
but still make it possible for users running a customized JupyterLab
distribution without a status bar to use the consumer plugin.

More Details about Tokens
"""""""""""""""""""""""""

JupyterLab uses tokens to identify reusable features (remember, these are also called "service objects" [ADD_LINK])
in the extension system. A token is an instance of the Lumino Token class.

.. note::
   JupyterLab uses tokens to identify services (instead of strings, for
   example) to prevent conflicts between identifiers and to enable type
   checking when using TypeScript.

In JupyterLab's Provider-Consumer Pattern:

- A "provider" plugin will list its token in the plugin metadata ``provides``
field, and will return the associated service object from its ``activate``
function.

- "Consumer" plugins will import the token: For example, the token might be
imported from one of these places:

  - The Javascript package that the provider plugin comes from
  - From a third package that exports the token for use by both the provider and
    the consumer (this pattern is commonly used by JupyterLab)

The consumer plugin will then list the token in their plugin metadata
``requires`` or ``optional`` fields.

When your extension is loaded by JupyterLab, any services provided by your
plugins (which have been exported) are registered with the extension system.

A token defined in TypeScript can also provide a TypeScript interface for
the service it is associated with, to allow for extra type-checking (if a
package using the token uses TypeScript, the service will be type-checked
against this interface when the package is compiled to JavaScript). This
can help prevent errors by ensuring that a service cannot be swapped out
unless it is compatible with the original service object.

The status bar in JupyterLab is an example of a service that your plugins
can request, which allows you to add your own status bar items, and it is
identified by the IStatusBar token.

To request the status bar service object, you need to import the IStatusBar
token, and then list it in the ``requires`` or ``optional`` field of your
plugin. JupyterLab will then pass the status bar service object as an argument
to your plugin's ``activate`` function when it loads your extension (if it's
available).

Making Your Plugin a Provider
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To make your plugin a "provider" of service objects that other plugins can use,
you need to list a ["Token"](https://lumino.readthedocs.io/en/latest/api/classes/coreutils.Token.html#constructor)
in your plugin's "provides" property, then return an object from your plugin's
``activate`` function.

Take a look at a snippet from [this example extension](https://github.com/jupyterlab/extension-examples/tree/main/step_counter)
in the examples repo (you can read the full extension example code there):

.. code::

  // This plugin is a "provider" in JupyterLab's provider-consumer pattern.
  // For a plugin to become a provider, it must list the token it wants to
  // provide a service object for in its "provides" list, and then it has
  // to return that object (in this case, an instance of the example Counter
  // class defined above) from the function supplied as its activate property.
  // It also needs to supply the interface (the one the service object
  // implements) to JupyterFrontEndPlugin when it's defined.
  const plugin: JupyterFrontEndPlugin<StepCounterItem> = {
    id: 'step_counter:provider_plugin',
    description: 'Provider plugin for the step_counter\'s "counter" service object.',
    autoStart: true,
    provides: StepCounter,
    // The activate function here will be called by JupyterLab when the plugin loads
    activate: (app: JupyterFrontEnd) => {
      console.log('JupyterLab extension (step_counter/provider plugin) is activated!');
      const counter = new Counter();

      // Since this plugin "provides" the "StepCounter" service, make sure to
      // return the object you want to use as the "service object" here (when
      // other plugins request the StepCounter service, it is this object
      // that will be supplied)
      return counter;
    }
  };

Here, you can see that this plugin lists a "StepCounter" token object as its
"provides" property, which tells JupyterLab that it is a "provider" of a
service object.

It also returns a "Counter" instance: this is the service object it "provides"
for the StepCounter service.

When your plugin becomes a provider, you need to define a Lumino "Token" object
that JupyterLab will use to identify the service. Here's how the StepCounter
Token was defined:

.. code::

  // The token is used to identify a particular "service" in
  // JupyterLab's extension system (here the StepCounter token
  // identifies the example "Step Counter Service", which is used
  // to store and increment step count data in JupyterLab). Any
  // plugin can use this token in their "requires" or "activates"
  // list to request the service object associated with this token!
  const StepCounter = new Token<StepCounterItem>(
    'step_counter:StepCounter',
    'A service for counting steps.'
  );

Note that StepCounter is a Lumino Token object. The StepCounter defined
here also passes the "StepCounterItem" interface in the Token definition.

When you provide an interface to your Token definition in this way, you're
telling JupyterLab to type check the service object it gets from any provider
plugin associated with this service, to make sure it conforms to that
interface. This helps ensure that any provider plugin (even a substitute
provider that someone else makes later) provides a compatible service object
(in this case, a StepCounterItem object), and it helps enable the plugin
swappability and subsitution in JupyterLab.

Here's the interface the token uses:

.. code::

  // The StepCounterItem interface is used as part of JupyterLab's
  // provider-consumer pattern. This interface is supplied to the
  // token instance (the StepCounter token), and JupyterLab will
  // use it to type-check any service-object associated with the
  // token that a provider plugin supplies to check that it conforms
  // to the interface.
  interface StepCounterItem {
    // registerStatusItem(id: string, statusItem: IStatusBar.IItem): IDisposable;
    getStepCount(): number;
    incrementStepCount(count: number): void;
    countChanged: Signal<any, number>;
  }

This means that anyone who makes a provider plugin for the StepCounter service
must return an object that has a getStepCount method, incrementStepCount method,
and a countChanges Signal (a Lumino Signal object).

Plugin Settings
^^^^^^^^^^^^^^^

JupyterLab includes a plugin settings system that can be used to provide default
setting values and user overrides. A plugin's settings are specified with a
JSON schema file. The ``jupyterlab.schemaDir`` field in ``package.json``gives
the relative location of the directory containing plugin settings schema files.

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






Structuring Your Extension
--------------------------

Now that you understand the plugin system, FOOBAR

Designing for Reusability
"""""""""""""""""""""""""

If your extension has a provider plugin with an exported token, consumers
will need to import that token to use it. That token should be made available
by exporting it in a JavaScript package that consumers can import and use.
Tokens need to be deduplicated in JupyterLab, and there are tools to do this
for you, which you can read about in the :ref:`deduplication` section.

A commonly used pattern in JupyterLab is to create and export a token from
a standalone package that both the provider and consumer extensions import
(instead of defining the token in the provider's package). This empowers a
user to swap out the provider plugin for a different plugin that provides
the same token, but with an alternative service implementation.

For example:

- The core JupyterLab ``filebrowser`` package exports a token
  which represents the file browser service (enabling interactions with the
  file browser)

- The ``filebrowser-extension`` package contains a plugin that implements
  the file browser in JupyterLab, and provides the file browser service to
  JupyterLab (identified with the token imported from the ``filebrowser``
  package)

Using this pattern, extensions in JupyterLab that want to interact with the
filebrowser-extension do not need to have a JavaScript dependency on the
``filebrowser-extension`` package: They only need to import the token from
the ``filebrowser`` package. This pattern enables users to seamlessly change
the file browser in JupyterLab by writing their own extension that imports
the same token from the ``filebrowser`` package, and provides it to the
system with their own alternative file browser service.

..
   We comment out the following, until we can import from a submodule of a package. See https://github.com/jupyterlab/jupyterlab/pull/9475.

   A pattern in core JupyterLab is to create and export tokens from a self-contained ``tokens`` JavaScript module in a package. This enables consumers to import a token directly from the package's ``tokens`` module (e.g., ``import { MyToken } from 'provider/tokens';``), thus enabling a tree-shaking bundling optimization to possibly bundle only the tokens and not other code from the package.

.. _schemaDir:



















Types of Plugins
^^^^^^^^^^^^^^^^

JupyterLab supports several types of plugins (some with extras restrictions and limitations):

-  **Application plugins:** Application plugins are the fundamental building block of JupyterLab functionality. Application plugins interact with JupyterLab and other plugins by requiring services provided by other plugins, and optionally providing their own service to the system. Application plugins in core JupyterLab include the main menu system, the file browser, and the notebook, console, and file editor components.
-  **Mime renderer plugins:** Mime renderer plugins are simplified, restricted ways to extend JupyterLab to render custom mime data in notebooks and files. These plugins are automatically converted to equivalent application plugins by JupyterLab when they are loaded. Examples of mime renderer plugins that come in core JupyterLab are the pdf viewer, the JSON viewer, and the Vega viewer.
-  **Theme plugins:** Theme plugins provide a way to customize the appearance of JupyterLab by changing themeable values (i.e., CSS variable values) and providing additional fonts and graphics to JupyterLab. JupyterLab comes with light and dark theme plugins.

Application Plugins
^^^^^^^^^^^^^^^^^^^

An application plugin is a JavaScript object with a number of metadata fields. A typical application plugin might look like this in TypeScript:

.. code-block:: typescript

   const plugin: JupyterFrontEndPlugin<MyToken> = {
     id: 'my-extension:plugin',
     description: 'Provides a new service.',
     autoStart: true,
     requires: [ILabShell, ITranslator],
     optional: [ICommandPalette],
     provides: MyToken,
     activate: activateFunction
   };

The ``id`` and ``activate`` fields are required and the other fields may be omitted. For more information about how to use the ``requires``, ``optional``, or ``provides`` fields, see :ref:`services`.

- ``id`` is a required unique string. The convention is to use the NPM extension package name, a colon, then a string identifying the plugin inside the extension.
- ``description`` is an optional string. It allows to document the purpose of a plugin.
- ``autostart`` indicates whether your plugin should be activated at application startup. Typically this should be ``true``. If it is ``false`` or omitted, your plugin will be activated when any other plugin requests the token your plugin is providing.
- ``requires`` and ``optional`` are lists of :ref:`tokens <tokens>` corresponding to services other plugins provide. These services will be given as arguments to the ``activate`` function when the plugin is activated. If a ``requires`` service is not registered with JupyterLab, an error will be thrown and the plugin will not be activated.
- ``provides`` is the :ref:`token <tokens>` associated with the service your plugin is providing to the system. If your plugin does not provide a service to the system, omit this field and do not return a value from your ``activate`` function.
- ``activate`` is the function called when your plugin is activated. The arguments are, in order, the :ref:`application object <application_object>`, the services corresponding to the ``requires`` tokens, then the services corresponding to the ``optional`` tokens (or ``null`` if that particular ``optional`` token is not registered in the system). If a ``provides`` token is given, the return value of the ``activate`` function (or resolved return value if a promise is returned) will be registered as the service associated with the token.

.. _rendermime:

MIME Renderer Plugins
^^^^^^^^^^^^^^^^^^^^^

MIME Renderer plugins are a convenience for creating a plugin
that can render mime data in a notebook and files of the given mime type. MIME renderer plugins are more declarative and more restricted than standard plugins.
A mime renderer plugin is an object with the fields listed in the
`rendermime-interfaces IExtension <../api/interfaces/rendermime_interfaces.IRenderMime.IExtension.html>`__
object.

JupyterLab has a `pdf mime renderer extension <https://github.com/jupyterlab/jupyterlab/tree/main/packages/pdf-extension>`__, for example. In core JupyterLab, this is used to view pdf files and view pdf data mime data in a notebook.

We have a `MIME renderer example <https://github.com/jupyterlab/extension-examples/tree/master/mimerenderer>`__  walking through creating a mime renderer extension which adds mp4 video rendering to JupyterLab. The `extension template <https://github.com/jupyterlab/extension-template>`_ supports MIME renderer extensions.

The mime renderer can update its data by calling ``.setData()`` on the
model it is given to render. This can be used for example to add a
``png`` representation of a dynamic figure, which will be picked up by a
notebook model and added to the notebook document. When using
``IDocumentWidgetFactoryOptions``, you can update the document model by
calling ``.setData()`` with updated data for the rendered MIME type. The
document can then be saved by the user in the usual manner.

Theme plugins
^^^^^^^^^^^^^

A theme is a special application plugin that registers a theme with the ``ThemeManager`` service. Theme CSS assets are specially bundled in an extension (see :ref:`themePath`) so they can be unloaded or loaded as the theme is activated. Since CSS files referenced by the ``style`` or ``styleModule`` keys are automatically bundled and loaded on the page, the theme files should not be referenced by these keys.

The extension package containing the theme plugin must include all static assets that are referenced by ``@import`` in its theme CSS files. Local URLs can be used to reference files relative to the location of the referring sibling CSS files. For example ``url('images/foo.png')`` or ``url('../foo/bar.css')`` can be used to refer local files in the theme. Absolute URLs (starting with a ``/``) or external URLs (e.g. ``https:``) can be used to refer to external assets.

See the `JupyterLab Light Theme <https://github.com/jupyterlab/jupyterlab/tree/main/packages/theme-light-extension>`__ for an example.

See the `TypeScript extension template <https://github.com/jupyterlab/extension-template>`__ (choosing ``theme`` as ``kind`` ) for a quick start to developing a theme plugin.

















______________


In the following discussion, the plugin that is providing a service to the
system is the *provider* plugin, and the plugin that is requiring and using
the service is the *consumer* plugin. Note that these kinds of *provider*
and *consumer* plugins are fundamental parts of JupyterLab's Provider-Consumer
pattern (which is a type of `dependency-injection <https://en.wikipedia.org/wiki/Dependency_injection>`_
pattern).










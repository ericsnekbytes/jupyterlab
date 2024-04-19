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

Let's take a very quick look at a basic "Application Plugin" (you will learn
extra details about application plugins later):

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

So, this application plugin is a TypeScript object with several metadata
fields. The ``id`` and ``activate`` fields are mandatory, the others are
optional.

At the highest level, JupyterLab will load your extension, it will read
information from these fields, and will then run the ``activate`` function
you supplied in your ``activate`` field. You will use the ``activate``
function to perform the necesssary setup steps to launch your extension.

A more detailed explanation of these fields is included later in this
document, along with details about the different kinds of plugins, and what
they are used for.

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

This design scheme is called the ["Provider-Consumer Pattern"](ADD_LINK) in
JupyterLab, and is a type of [dependency injection pattern](ADD_LINK).

.. note::
    Read more about the general concept of software "design patterns" here.

JupyterLab's Provider-Consumer Pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Provider-Consumer Pattern is the heart of what powers the reusability,
extensibility, and swappability of core parts of the JupyterLab application,
and of the extension system as a whole. Its name comes from the fact that
one plugin "provides" a service object to the system, and other plugins can
"consume" that object to reuse the features it provides.

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

How Requesting Features Works
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you designate a feature in the ``requires`` list of your plugin, JupyterLab
will only load your plugin if that feature is available (it will fail to load
otherwise).

By designating a feature in the ``optional`` list, JupyterLab will
pass you an object for it (if it's available) or ``null`` if it's not.

Both of these behaviors can be used to enable compatibility with multiple
Jupyter applications (like JupyterLab + Jupyter Notebook 7), which you can
read more about in the :ref:`Compatibility Guide <extension_dual_compatibility>`.

In order to identify a particular service object/feature, JupyterLab uses
"Token" objects (instances of the Lumino Token class [ADD_LINK]), so the items
that you list in your ``requires`` and ``optional`` fields are not actually
the service objects or the plugins themselves, they are actually token objects
that are used to identify and fetch the actual service objects they are
associated with.

Unless your plugins are providing services to the system, you probably won't
need to know much about Tokens: You can just think of them as identifiers.

JupyterLab orders plugin activation to ensure that a provider of a service
is activated before its consumers. A token can only be registered with the
system once.

A consumer might list a token as ``optional`` when the service it identifies
is not critical to the consumer, but would be nice to have if the service is
available. For example, a consumer might list the status bar service as
optional so that it can add an indicator to the status bar if it is available,
but still make it possible for users running a customized JupyterLab
distribution without a status bar to use the consumer plugin.

More Details about Tokens
"""""""""""""""""""""""""

When your extension is loaded by JupyterLab, any services from your plugins
which are exported are registered with the extension system.

To identify those services, JupyterLab uses a *token*, i.e., a concrete
instance of the Lumino Token class.

.. note::
   JupyterLab uses tokens to identify services (instead of strings, for
   example) to prevent conflicts between identifiers and to enable type
   checking when using TypeScript.

A "provider" plugin will list its token in the plugin metadata ``provides``
field, and will return the associated service object from its ``activate``
function.

"Consumer" plugins will import the token: For example, the token might be
imported from one of these places:

- The Javascript package that the provider plugin comes from
- From a third package that exports the token for use by both the provider and
  the consumer (this pattern is commonly used by JupyterLab)

The consumer plugin will then list the token in their plugin metadata
``requires`` or ``optional`` fields.

A token defined in TypeScript can also provide a TypeScript interface for
the service it is associated with, to allow for extra type-checking (if a
package using the token uses TypeScript, the service will be type-checked
against this interface when the package is compiled to JavaScript). This
can help prevent errors by ensuring that a service cannot be swapped out
unless it is compatible with the original service object.

Publishing Your Own Tokens
""""""""""""""""""""""""""

If your extension has a provider plugin with an exported token, consumers
will need to import that token to use it. That token should be exported in
a published JavaScript package. Tokens need to be deduplicated in JupyterLab,
and there are tools to do this for you, which you can read about in the
:ref:`deduplication` section.

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


























As a type of dependency injection pattern
Reusability Patterns in JupyterLab
The heart of reusability and 

one plugin "provides" an object (called a
**service object**) to the system, and other plugins "consume" that object



Plugins are a foundational piece of the larger [JupyterLab extension system](ADD_LINK),
make sure you read about that first here [ADD_LINK]).


By splitting up your features into multiple plugins, they can be reused by
other extensions, or swapped out so that users can easily modify small pieces
of your extension.


the fact that one plugin "provides" an object (called a
**service object**) to the system, and other plugins "consume" that object
by using it in an extension to add extra features and customizations
to JupyterLab (and an extension consists of one or more plugins).




In the following discussion, the plugin that is providing a service to the
system is the *provider* plugin, and the plugin that is requiring and using
the service is the *consumer* plugin. Note that these kinds of *provider*
and *consumer* plugins are fundamental parts of JupyterLab's Provider-Consumer
pattern (which is a type of `dependency-injection <https://en.wikipedia.org/wiki/Dependency_injection>`_
pattern).






Plugins are a foundational piece of the larger [JupyterLab extension system](ADD_LINK)
(read about that first here [ADD_LINK]),

Plugins are a foundational piece of the larger [JupyterLab extension system](ADD_LINK)
(read about that first here [ADD_LINK]), so it's helpful to start by stating
that each JupyterLab extension is a package that contains one or more plugins.

Plugins are a foundational piece of the [JupyterLab extension system](ADD_LINK)
(read more about that here), which gives you the power to build new features
for JupyterLab.




















JupyterLab's plugin system is designed so that plugins can depend on and
reuse features from one another. A key part of this approach is JupyterLab's
**provider-consumer pattern**.

The Provider-Consumer Pattern
-----------------------------

..
    TODO add to glossary, provider-consumer, service objects, tokens

In the provider-consumer pattern, one plugin "provides" an object (called a
**service object**) to the system, and other plugins "consume" that object
by using it in an extension to add extra features and customizations
to JupyterLab (and an extension consists of one or more plugins).
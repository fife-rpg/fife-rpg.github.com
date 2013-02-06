.. _components:
.. include:: ..\include.inc

Components
==========

This tutorial will explain the component system FIFErpg uses and show
how to create your own components.

bGrease
-------
FIFErpg uses bGrease, which is a fork of the the
`Grease Framework <http://packages.python.org/grease/index.html>`_.
Grease components are simple classes that only contain information about
what fields a component has.
Adding a grease component is as simple as putting this code inside the
configure method of a grease world:

.. code :: python

   self.components.health = component.Component(
       health=int, max_health=int)


FIFErpg
-------

While this would also perfectly work with FIFErpg this does not use the auto
registering functions. For that you would need to make your own component
class, that inherits from the :py:class:`~fife_rpg.components.base.Base`
component class.

First we will need to import the Base class.

.. code :: python

   from fife_rpg.components.base import Base

Then make a class that derives from that.

.. code :: python

   class Health(Base):

To define the fields you will need to call the Base class constructor with
their information:

.. code :: python

   def __init(self):
      Base.__init__(health=int, max_health=int)

The possible types for the fields can be found in the components
`documentation <http://beliaar.github.com/bGrease/mod/component.html#bGrease.component.Component>`_.

If you want to set the default value for a field you need to add this in the
constructor:

.. code :: python

   self.fields["health"].default = lambda: 100
   self.fields["max_health"].default = lambda: 100
   
You can also set the default to a type that can be constructed with no
arguments, you won't need the lambda then.

Now, the next important part is the "register" class method, which needs to be
overwritten. The following code is all that is needed for that:

.. code :: python

    @classmethod
    def register(cls, name="Health", auto_register=True):
        return (super(Health, cls).register(name, auto_register))

Dependencies and inheriting from another component
--------------------------------------------------
If you want to have the component automatically register other components,
or any of the other FIFErpg classes the support auto registering. You can
set the dependencies class member. It is a list of classes. When registering
a component the register method will look through all the classes in that
list and calls their register method if the class is not already registered.
This is done before registering the component itself.

Also, if the component derives from a component that is not the Base component,
the register method will, by default, make it so that the class that the
component derives from will appear as registered but points to the component
that derives from it. This can be used to make components that replace other
components. This can be disabled by setting the "auto_register" argument
of the register method to False.

Component functions
-------------------
While components only store data and have no methods attached that work with
them, the modules of the default components have functions in them that do
various things with the components. They don't have any specific format, so
when you want to write your own functions you can write them however you need
them to be.

Script commands
---------------
The component manager, to which every components registers itself when the
register method is called, can also be used to register script commands.
These commands will automatically be available to scripts if the scripting
system is activated.

You may need to import the manager, this is done with this code:

.. code :: python

   from fife_rpg.components import ComponentManager
   
Then to register a command:

.. code :: python

   ComponentManager.register_script_command(<command_name>, <function>)

When registering the components to the application one can disable the
auto registering of script commands. For this to work the register commands
have to be inside a "register_script_commands" function, inside the same module
as the component, which takes no arguments.

Checkers
--------
Another feature, which the component manager has, is to register
checker functions. These functions should look for inconsistencies in
components and, if possible, fix them, e.g. for the lockable component that an
possibly attached fifeagent is playing the appropriate animation. The functions
are called every pump/tick by the world.

As for component functions what the functions actually do is up to you.

Registering such a function is done as follows:

.. code :: python

   ComponentManager.register_checker(
       <list of component names>,
       <checker function>)
       
As with the script commands you should these functions inside a
"register_checkers" function inside the same module as the component.
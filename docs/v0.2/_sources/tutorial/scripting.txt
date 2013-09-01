.. _scripting:

.. include:: ..\include.inc

Scripting
=========

This tutorial will explain the scripting system of FIFErpg.

.. note::

   Scripting has changed since the 0.1c release.

Overview
--------
The scripting system keeps a dictionary of scripts and runs specific functions
on specific events.

Scripting System
----------------
The |ScriptingSystem| itself manages a dictionary of scripts.

Scripts
-------
A Script is a python file that is being run inside the scripting environment.

The scripts need to have specific functions that are being called by the
scripting system for them to actually do something after the initial run.

Currently there are 2 functions that the scripting system calls, if a script
has them.

step(time_delta)
This is called every time the scripting systems step method is called.
The time_delta is the time since the last call to step.

on_map_switched(self, old_map, new_map)
This is called after the map was switched. The variables have been updated for
the new map at this point. The old_map is the name of the previous map, the
new_map is the name of the current map.

Note that there are NO restrictions on what modules can be imported and used.
All modules that are available to the python environment in which the main
application is run are also available in scripts by importing them normally.

The additional functions and variables available are those that have been made
available to the |GameVariables| system and the commands that have been
registered to the |ScriptingSystem|. These are inside various modules,
depending on how they were registered or made available.

These are the current variables and modules available. Note that some may only
be available if the component or system is registered and active.

application - Functions and variables from the |Application|
   - current_map: :attr:`~fife_rpg.rpg_application.RPGApplication.current_map`
   - is_agent_in_region: :meth:`~fife_rpg.rpg_application.RPGApplication.is_agent_in_region`
   - is_location_in_region: :meth:`~fife_rpg.rpg_application.RPGApplication.is_location_in_region`
   - maps :attr:`~fife_rpg.rpg_application.RPGApplication.maps`
   - set_global_lighting: :meth:`~fife_rpg.rpg_application.RPGApplication.set_global_lighting`
   - get_global_lighting: :meth:`~fife_rpg.rpg_application.RPGApplication.get_global_lighting`

entities - Makes every |RPGEntity| of the |RPGWorld| available by its name.
For example you just need to type entities.PlayerCharacter

Note for components:
These are only the default names. When auto registering script commands
through the |Application| the name under which the component is registered
is used.

Agent - Functions of the Agent component.
   - knows : :func:`~fife_rpg.components.agent.knows`
   - add_knowledge : :func:`~fife_rpg.components.agent.add_knowledge`

FifeAgent - Functions of the FifeAgent component
   - move : :func:`~fife_rpg.components.fifeagent.approach_and_execute`

Adding Scripts
--------------
To add a script to the system you will need to use the :py:func:`add_script()
<fife_rpg.systems.scriptingsystem.ScriptingSystem.add_script()>` method.
It takes the scripts name and the path to the script file.

Example:

.. code:: python

   script_system.add_script("Test", "scripts/test.py")

It is also possible to add scripts YAML file.
The structure of the file is a list where each item has 2 values. The first
value is the name of the script, the second the path to the script file.

Example:

.. code:: yaml

   Scripts:
     - [test, scripts/test.py]


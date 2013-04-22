.. _game_variables:

.. include:: ..\include.inc

GameVariables
=============

This tutorial will explain the |GameVariables| system of FIFErpg.

Overview
--------
The |GameVariables| system manages game variables. These can be used for whatever
you want.

Variables
---------
Internally the system distinguishes between static and dynamic variables.
Dynamic variables can be freely set to any value you want. The values of static
variables can not be changed, although a dynamic variable with the same name
will hide the static variable.

Setting, reading and deleting variables
---------------------------------------
To set a dynamic variable use the
:py:func:`set_variable() <fife_rpg.systems.game_variables.GameVariables.set_variable>`
method. It accepts three arguments: name, value and allow_static_hide.
Name is the name of the variable, any previously variable with that name will
be overwritten. Value is the value of the variable. If allow_static_hide is
set to True it will allow the setting of a dynamic variable that has the same
name as an already present static one. The static variable will not be deleted,
only not accessible. allow_static_hide is set to False by default.

To get the value of a variable, both dynamic and static, use the
:py:func:`get_variable() <fife_rpg.systems.game_variables.GameVariables.get_variable>`
method. It accepts the name of the variable and returns the value.

To delete a dynamic variable use the
:py:func:`delete_variable() <fife_rpg.systems.game_variables.GameVariables.delete_variable>`
method. It accepts the name of the variable.

To get a combined dictionary of the current static and dynamic variables use the
:py:func:`get_variables() <fife_rpg.systems.game_variables.GameVariables.get_variables>`
method.

Adding callback functions for static variables
----------------------------------------------
The static variables are gathered from different parts of an application which
register callback functions to the system.

To add a callback function us the
:py:func:`add_callback() <fife_rpg.systems.game_variables.GameVariables.add_callback>`
method. It accepts a variable that should be callable. The callable should
accept a single argument which is a dictionary. It should then add its variables
to the dictionary. Anything that it returns will be ignored.

Console
-------
The system registers several commands to the console:

* SetVariable - Sets the specified variable to the specified value
* GetVariable - Returns the value of the specified variable
* DeleteVariable - Deletes the specified variable

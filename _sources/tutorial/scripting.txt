.. _scripting:

.. include:: ..\include.inc

Scripting
=========

This tutorial will explain the scripting system of FIFErpg.

Overview
--------
The scripting system evaluates a condition and runs the |Action| list of a
|Script| if it evaluates to True.

Scripts
-------
A |Script| contains an list of dictionaries, each containing data to create an
|Action|. Scripts are not to be created manually, rather use the methods
of the |ScriptingSystem| explained below.

The dictionaries can have the following keys, most are the same as for the
constructor of the |Action| class:

- Action: The name of the |Action|
- Agent: The performer of the |Action|
- Target: The target of the |Action|
- Time: The seconds to wait after the current action is finished before the
  next |Action| should be executed.
- ActionCommands: The additional commands the |Action| should execute.(optional)
- Command: This will pass the execution of the action over to a script command.
  (optional). The value is also a dictionary:

  - Name: The name of the command
  - Variables: A dictionary of variables to pass to the command.

   - Type: The Type of the variable. (optional) Currently only accepting
     "Entity", which interprets the value as the name of an Entity
   - Value: The value of the variables.

Scripting System
----------------
The |ScriptingSystem| itself manages a dictionary of scripts and a list of
conditions. Every time its :py:func:`step() <fife_rpg.systems.scriptingsystem.ScriptingSystem.step()>`
method is called it will iterate through the
list of conditions and evaluate them. If the evaluation returns True it will
run the script set by the condition.

To add a script to the system you will need to use the :py:func:`set_script()
<fife_rpg.systems.scriptingsystem.ScriptingSystem.set_script()>` method.
It takes the scripts name and the dictionary list as explained in the Scripts
section above.

Example:

.. code:: python

   action = {}
   action["Action"] = "MoveAgent"
   action["Agent"] = "PlayerCharacter"
   action["Target"] = "ToLevel2"
   action["Time"] = 0
   command = {}
   command["Name"] = "move"
   command["Variables"] = []
   command["Variables"].append({"Type": "Entity", "Value": "PlayerCharacter"})
   command["Variables"].append({"Type": "Entity", "Value": "ToLevel2"})
   action["Command"] = command
   script = [action]
   script_system.set_script("Test", script)

To add a condition to the system you will need to use the :py:func:`add_condition()
<fife_rpg.systems.scriptingsystem.ScriptingSystem.add_condition()>` method. It
takes a condition_data argument that is a dictionary with 2 values:

- Script: The name of the script of the condition passes
- Expressions: A list of expressions that form the condition. All expressions
  must evaluate to True for the condition to pass.
  Each expression itself is a dictionary with 2 values:

  - Type: The type of the condition. What conditions are available depends on
    what conditions where registered to the scripting system. The type can be
    prefixed with "Not\_" to negate its result.
  - Args: A list of arguments to pass to the condition. This depends on the
    condition.

Example:

.. code:: python

   expression = {}
   expression["Type"] = "Not_Knows"
   expression["Args"] = ["David", "PlayerCharacter"]
   condition = {}
   condition["Script"] = "Test"
   condition["Expressions"] = [expression]
   script_system.add_condition(condition)

It is possible to run a script manually by using the :py:func:`run_script()
<fife_rpg.systems.scriptingsystem.ScriptingSystem.run_script()>` method.
It takes the name of the script as an argument.

It is also possible to add scripts and conditions from a YAML file.
The structure of the file is a dictionary with 2 keys: Scripts and Conditions.
The structure of each key is the same as explained above.

Example:

.. code:: yaml

   Scripts:
       Test:
           - Action: MoveAgent
             Agent: PlayerCharacter
             Target: ToLevel2
             Time: 0
             Command:
               Name: move
               Variables:
                 - Type: Entity
                   Value: PlayerCharacter
                 - Type: Entity
                   Value: ToLevel2               
   Conditions:
     - Script: Test
       Expressions:
         - Type: Not_Knows
           Args: [David, PlayerCharacter]
    
    
.. _dialogues:

.. include:: ..\include.inc

Dialogues
=========

This tutorial will explain the dialogue module that is included in FIFErpg.

.. note::

   The conditions section has changed since the 0.1c release.

Summary
-------

Here is a quick summary of how the dialogues in this module work.

1. The dialogue is initiated and the first greeting which condition passes
   is selected as the current section.
2. The commands of the current section are executed.
3. If there are responses available go to 4. Otherwise go to 5.
4. Select a response as the current section. Go to 2.
5. End the dialogue.

The Dialogue Controller
-----------------------

The dialogue module has a basic dialogue controller. It does not do much by
itself. Its constructor either takes a |Dialogue|, a dictionary with the
dialogue data or a string that points to a yaml file with the dialogue data.
It provides easy access to the important methods and properties of the dialogue
and deactivates itself if the dialogue is finished.

The dialogue controller has, at the moment, no view associated with it which
leaves it up to you how dialogues are presented.

Dialogue Structure
------------------

The structure of the dictionary and the yaml file are basically the same:

There are two main sections: Greetings and Sections. Greetings is a list and
Sections is a dictionary but both basically store the same data.
The order of the items in Greetings determines the order in which the items are
evaluated. The first item which condition passes, from the top, is selected
as the greetings that is used. That means that general greetings should be
put at the bottom of the list and special ones at the top.

The name of the key in Sections is the name used to reference the section.

The rest of the structure is the same for both:

talker: The identifier of the entity that is talking.
text: The text that the entity is saying.
conditions: A list of conditions that need to pass for the section to be able to
be selected. The structure will be explained below.
(Optional. If not present or empty the section will always be able to be selected)
commands: A list of commands that are executed when the section is run. The
structure will be explained below. (Optional)
respones: A list of section names that can be selected if their conditions pass.
(Optional. If not present or empty the dialogue will finish after running the
current section)

Text
----
The text allows one to insert the values of variables. All variables that
the Game Variables systems has access to can be accessed as well. By default
all entities can be accessed, for example. To access the variables put them
inside curly brackets e.g. : {PlayerCharacter.Description.view_name}. There
is a special variables that is only available for dialogues: "DialogueTalker"
which points to the entity that is set with "talker" to allow easy access to
that entities values without having to retype the identifier and changing that
every time the identifier is changed.

Conditions
----------
The conditions themselves are an python expression that should evaluate
to either True or False. The expression is evaluated using the
|ScriptingSystem| thus the same variables and functions that scripts
have available are available to dialogue conditions.

See also:
:doc:`scripting`

Commands
--------
The commands also are dictionaries with 2 keys:

Name: The command to execute. All registered console commands are accepted.
Args: A list of arguments to pass to the command. This supports the same
variable replacement as for the text.

Here is an example dialogue file:

.. code:: yaml

   Greetings:
     
     - talker: David
       text: Hey, I've seen you before, haven't I?
       conditions: Agent.knows(entities.David, "PlayerCharacter")
       responses:
         - "yes"
         - yes_bye
             
     - talker: David
       text: Hi, I don't know you, do I?
       conditions: not Agent.knows(entities.David, "PlayerCharacter")
       responses:
         - no_introduce
         - no_bye
   
   Sections:
     no_introduce:
       talker: PlayerCharacter
       text: No. I am {DialogueTalker.Description.view_name}
       commands:
           - Name: AddKnowledge
             Args: [David, PlayerCharacter]
       responses:
         - introduce_reply
     no_bye:
       talker: PlayerCharacter
       text: No, and I don't want to change that.
     "yes":
       talker: PlayerCharacter
       text: Yes, I am {DialogueTalker.Description.view_name}. Remember?
       responses:
         - yes_reply
     yes_bye:
        talker: PlayerCharacter
        text: Yes, but I am in a hurry. Bye.
     introduce_reply:
       talker: David
       text: Nice to meet you, my name is {DialogueTalker.Description.real_name}
       commands:
         - Name: SetComponentValue
           Args: [David, Description, view_name, "{DialogueTalker.Description.real_name}"]
         - Name: AddKnowledge
           Args: [PlayerCharacter, David]
     yes_reply:
       talker: David
       text: Ah, right.
       
Accessing dialogue data and selecting responses
-----------------------------------------------
The easiest method to access the data of a dialogue is to create a
|DialogueController| and then use the methods provided to access the dialogue.

.. code:: python

   from fife_rpg.dialogue import DialogueController
   
   dlg = DialogueController(world, application, "dialogue/example.yaml")

To access the text:

.. code:: python

   dlg.current_section.text

To get the list of the responses which conditions pass:

.. code:: python
   
   responses = dlg.possible_responses

To select and run a response section:

.. code:: python

   dlg.select_response("no_introduce")

To check if the dialogue is finished:

.. code:: python
   
   if dlg.is_dialogue_finished:
      ...

Here is an example of a simple dialogue using the example file above. Normally
you would show the possible responses to the player and wait for them to select
a response, of course:

.. code:: python

   print dlg.current_section.text
   dlg.select_response("no_introduce")
   print dlg.current_section.text                         
   dlg.select_response("introduce_reply")
   print dlg.current_section.text     
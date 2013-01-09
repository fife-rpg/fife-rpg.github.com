.. _controlling:

Accessing and controlling entities
==================================
Now that we have entities we will want to access them and control them. This
tutorial will show how that can be done.

.. note::

   From now on I will not explain how to activate Components or
   Behaviours.

Getting an entity
-----------------

Of course the first thing is to actually get the entity. This can simply be done
with the :py:meth:`get_entity() <fife_rpg.world.RPGWorld.get_entity>` method
of the world. It takes the identifier of the entity, which is set in the
identifier field of the General component. For the character from our last
tutorial it is "PlayerCharacter". So to get this entity you need to add this
line of code after the world and the entities have been created:

.. code-block:: python

   player = world.get_entity("PlayerCharacter")
   
This will store the entity in the "player" variable.

The components and fields of each component can be accessed like a member
variable.

Example:

.. code-block:: python
   
   print player.Agent.position
   
This should print "[-5, 0]" to the console.

You can also set the component fields:

.. code-block:: python
   
   player.Agent.position = [0, 7] 

Note that changing the fields does not necessarily have an immediate effect.
Also the new value will, in most cases, not be validated automatically.
If it is safe to directly modify the values depends on the actual component.

Component functions
-------------------

The component modules can have functions that work with entities that have
that component.

The fifeagent module, for example, has functions that can used to make the
agent walk or run to a location or another agent. However the entity needs
to have another component which stores the walk and run speed and animations:
The Moving component. For the object used in the tutorial we only need to set
the speeds, as the default values for the animations can be used with it.

For the tutorial we set the fields to the following values:

.. code-block:: yaml

  Moving:
      walk_speed: 1
      run_speed: 4

We will use the :py:func:`run() <fife_rpg.components.fifeagent.run>` function,
which takes two arguments, entity and location. The entity is a
:py:class:`RPGEntity <fife_rpg.entities.rpg_entity.RPGEntity>` and the location
can be a `fife.Location <http://www.fifengine.net/epydoc/fife.fife.Location-class.html>`_
or a position tuple (Like (0, 7)).

Example:

.. code-block:: python
   
   fifeagent.run(player, (0, 7))
   
The import line for fifeagent is:

.. code-block:: python

   from fife_rpg.components import fifeagent
   
If you add the example code after switching to "Level1" the entity will run to
the right.

A practical example
-------------------
To close this part of the tutorial off we will look at a practical example of
what we just learned: Using the mouse to let an entity move to a position.

For that we first need a custom Listener for the GameSceneController.

Create a new module and add the following imports:

.. code-block:: python

   from fife import fife
   
   from fife_rpg.game_scene import GameSceneListener
   from fife_rpg.components import fifeagent
   
A Listener is a class that receives events from FIFE. The default
GameSceneListener already listens for mouse events , so we only need to
override the event method that we want.

For other events, such as key presses, you would need to derive from the
respective fife listener and add that listener to the event manager when
the listener is activated, and remove it when the listener is deactivated.
Also you need to add all possible events from the fife listener or you will
get errors.

If we would want to add key events, for example, we would need the following
definition:

.. code-block:: python

   class Listener(GameSceneListener, fife.IKeyListener):
   
       def __init__(self, engine, gamecontroller=None):
           GameSceneListener.__init__(self, engine, gamecontroller)   
           fife.IKeyListener.__init__(self)                
      
       def activate(self):
           GameSceneListener.activate(self)
           self.eventmanager.addKeyListener(self)
   
       def deactivate(self):
           GameSceneListener.deactivate(self)
           self.eventmanager.removeKeyListener(self)

       def keyPressed(self, event):
           pass
   
       def keyReleased(self, event):
           pass

self._eventmanager.addMouseListener will add our Listener as a
KeyListener and removeKeyListener will remove it.
The activate, and deactivate methods are called by the GameSceneController when
it gets activated and deactivated, respectively.

We only need to override the mousePressed method, though, with the following code:

.. code-block:: python

    def mousePressed(self, event):
        player = self._application.world.get_entity("PlayerCharacter")
        if event.getButton() == fife.MouseEvent.LEFT:
            scr_point = self._application.screen_coords_to_map_coords(
                            (event.getX(), event.getY()), "actors"
                            )
            fifeagent.run(player, scr_point)

So our listener looks like this:

.. code-block:: python

   class Listener(GameSceneListener):
       
       def mousePressed(self, event):
           player = self.gamecontroller.application.world.get_entity("PlayerCharacter")
           if event.getButton() == fife.MouseEvent.LEFT:
               scr_point = self.gamecontroller.application.screen_coords_to_map_coords(
                               (event.getX(), event.getY()), "actors"
                               )
               fifeagent.run(player, scr_point)
            
First we get the entity with the identifier "PlayerCharacter".
Then we check if it was the left mouse button that was clicked with, as we only
want to react to clicks with that button.
Next we use the applications :py:meth:`screen_coords_to_map_coords() <fife_rpg.rpg_application.RPGApplication.screen_coords_to_map_coords>`
method which takes a `fife.ScreenPoint <http://www.fifengine.net/epydoc/fife.fife.ScreenPoint-class.html>`_
or a tuple of a position on the screen and the name of a layer and returns
the matching coordinates on given layer of the current map as a `fife.Location <http://www.fifengine.net/epydoc/fife.fife.Location-class.html>`_.
We give it the location of the mouse click.
Last we call the fifeagent.run function with the entity and the converted
coordinates.

Now we need to modify the main module to use our Listener instead of the
default one. Just replace "controller = GameSceneController(view, app)" with
"controller = GameSceneController(view, app, listener=Listener(app.engine))",
and import the module containing the listener.

If you run the code now the entity should run to the location on the map you
click on.

Here is the complete code for this tutorial:

tutorial_scene.py

.. code-block:: python
   :emphasize-lines: 1-

   from fife import fife
   
   from fife_rpg.game_scene import GameSceneListener
   from fife_rpg.components import fifeagent
   
   class Listener(GameSceneListener):
       
       def mousePressed(self, event):
           player = self.gamecontroller.application.world.get_entity("PlayerCharacter")
           if event.getButton() == fife.MouseEvent.LEFT:
               scr_point = self.gamecontroller.application.screen_coords_to_map_coords(
                               (event.getX(), event.getY()), "actors"
                               )
               fifeagent.run(player, scr_point)
           
run.py

.. code-block:: python
   :emphasize-lines: 7, 18, 25-26

   from fife_rpg import RPGApplication
   from fife_rpg import GameSceneView
   from fife_rpg import GameSceneController
   from fife.extensions.fife_settings import Setting
   from fife_rpg.components import fifeagent
   
   from tutorial_scene import Listener
   
   settings = Setting(app_name="Tutorial 4", settings_file="settings.xml")
   
   def main():
       app = RPGApplication(settings)
       app.load_components("combined.yaml")
       app.load_behaviours("combined.yaml")
       app.register_components()
       app.register_behaviours()
       view = GameSceneView(app)
       controller = GameSceneController(view, app, listener=Listener(app.engine))
       app.create_world()
       app.load_maps()
       world = app.world
       world.import_agent_objects()
       world.load_and_create_entities()
       app.switch_map("Level1")
       player = world.get_entity("PlayerCharacter")
       fifeagent.run(player, (0, 7))
       app.push_mode(controller)
       app.run()
   
   if __name__ == '__main__':
       main()

The :ref:`next <actions>` Tutorial will look at the actions system of FIFErpg.
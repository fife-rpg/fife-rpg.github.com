.. _actions:
.. include:: ..\include.inc

Context based actions
=====================
In this tutorial I will explain the actions system of FIFErpg, which
automatically creates a list of actions that one entity can perform with
another. Such a list could be used, for example, to populate a context menu -
which is, however, not part of this tutorial.

Preparations
------------
First we need to add a new component and an action to the settings.
The name of the component is "Description".
The actions are set in the "Actions" setting which works the same as
"Components" and "Behaviours" do. The only action we need is the "Look" action.

You can review the changes in the
:download:`settings <actions/settings-dist.xml>` file.

.. hidden-literalinclude:: actions/settings-dist.xml

We are also going to add another entity, on which we can perform the action.
In "entities.yaml" add a new yaml document with the following content:

.. code :: yaml

   !Entity
   Components:
     General:
         identifier: David
     Agent:
         gfx: player
         map: Level1
         layer: actors
         position: [-1, 0]
         rotation: 180
         behaviour_type: Base
     Moving:
         walk_speed: 1
         run_speed: 4
     Description:
         real_name: David
         view_name: David
         desc: This is David.
         
Compare the result with the :download:`entities <actions/objects/entities.yaml>`
file.

The entity is mostly exact like the PlayerCharacter with an "Description"
component added. This is needed for the "Look" action.

Code changes
------------
Activating the actions is done the same way as for components and behaviours:

.. code :: python

   app.load_actions("combined.yaml")
   app.register_actions()

The Look action returns a string when it is executed, thus we need a way to
display that. We are going to use the "say" method of the fife instance. For
that we first need to activate the "FloatingTextRenderer" for the layer the
instance is on.
A good way to do this is to activate it for all layers, that entities can be
on, when the map is switched.

For this we are going to make a custom GameSceneController.
In the same line where GameSceneListener is imported add GameSceneController
as well, so the line should look like this:

.. code :: python

   from fife_rpg.game_scene import GameSceneListener, GameSceneController
   
Also add the following import, I will explain what it is used for later:

.. code :: python

   from fife.extensions.pychan.internal import get_manager
   
The basic code for the Controller is as follows:

.. code :: python

   class Controller(GameSceneController):
       
       def __init__(self, view, application, outliner=None, listener=None):
           listener = listener or Listener(application.engine, self)
           GameSceneController.__init__(self, view, application, outliner, listener)

The first line in the constructor will set the listener to our custom Listener
unless a listener was passed. This mimics the GameSceneController.

Next thing we need to add is a method that gets called when the map is
switched:

.. code :: python

    def on_map_switched(self, old_map, new_map):
        renderer = fife.FloatingTextRenderer.getInstance(self.application.current_map.camera)
        font = get_manager().getDefaultFont()
        renderer.setFont(font)
        renderer.addActiveLayer(self.application.current_map.get_layer("actors"))
        renderer.setBackground(100, 255, 100, 165)
        renderer.setBorder(50, 255, 50)
        renderer.setEnabled(True)

The name of the method does not really matter, as it will be used as a
callback function.

First we get the instance of the FloatingTextRender of the camera of the
current map. Then we will use the get_manager method to get the pychan
manager which is used to get the default font of pychan.

Next we set the renderer to use this font and add the "actors" layer to its
active layers. We also set the background and border colour of the text window
and enable the renderer.

Now we just need to add this as a callback when the map has switched. This is
done by adding this line to the constructor of the controller:

.. code :: python

   application.add_map_switch_callback(self.on_map_switched)
   
Choosing a target
-----------------
To perform an action you need a target, on which the action is performed.
We will use the right mouse button to do this. But first we are going to better
make visible what entity the mouse is currently on. The instance renderer of
FIFE can be set to add outlines around an instance. In FIFErpg this can be
easily activated by setting the "is_outlined" property of the listener of
the game scene controller. This can be done by adding this line after
creating the controller:

.. code :: python

   controller.listener.is_outlined = True

By default the controller uses the :py:class:`SimpleOutliner() <fife_rpg.game_scene.SimpleOutliner>`.
The outliner can be access through the controller. If we want, for example, to
not have an outline around the player character we can add the following line:

.. code :: python

   controller.outliner.outline_ignore.append("PlayerCharacter")
   
Note that the outliner does only affect what entities get an outline, not which
can be clicked on.

Now, since we want to react to right clicks, we need to add code to the
mousePressed event of our listener.

First, though, since we need to access the application
a lot for what we are going to do, we store it in a variable. Replace the code
in the method with this:

.. code :: python

   application = self.gamecontroller.application
   player = application.world.get_entity("PlayerCharacter")
   if event.getButton() == fife.MouseEvent.LEFT:
      map_point = application.screen_coords_to_map_coords(
                      (event.getX(), event.getY()), "actors"
                      )
      fifeagent.run(player, map_point)

To check for right clicks we just need to add the same condition as for
left clicks with LEFT replaced by RIGHT, like this:

.. code :: python

   elif event.getButton() == fife.MouseEvent.RIGHT:
   
The first thing we need is to get all instances that are at the position of the
mouse click, which, mostly, should be one.

For that we add this code in the the block of the right mouse click:

.. code :: python

   game_map = application.current_map
   if game_map:
       scr_point = fife.ScreenPoint(event.getX(), event.getY())
       actor_instances = game_map.get_instances_at(
                                           scr_point, 
                                           game_map.get_layer("actors")
                                           )
   else:
       actor_instances = ()
       
For better readability we first store the current map of the application in
the game_map variable. Then we check if there is actually a map loaded. If
there is we create a ScreenPoint using the coordinates of the click event.
With that we get the instances at this position that are on the "actors" layer
and store them in a tuple variable named "actor_instances". If there is no map
we set the actor_instances variable to an empty.

Next we are going to iterate over the actor_instances tuple.

.. code :: python

   for actor in actor_instances:

Then we check if the identifier is not in the "outline_ignore" list of the
outliner and skip the instance if it is.

.. code :: python

   identifier = actor.getId()
   outliner = self.gamecontroller.outliner
   if identifier in outliner.outline_ignore:
       continue
     
Then we will use the
:py:meth:`get_possible_actions() <fife_rpg.actions.action_manager.get_possible_actions>`
method of the ActionManager to create a list of the actions that can be
performed on this instance by the player entity.

.. code :: python

    entity = application.world.get_entity(identifier)
    possible_actions = ActionManager.get_possible_actions(
                                                 player,
                                                 entity)
                                                 
The get_possible_actions method takes two entities, a performer and a target.
The performer is the entity doing the action and the target is the entity
on which the action is done. It uses the check_performer and ceck_target
class methods of the registered actions and returns true if both evaluate
to true and false if one or both of them does not.

The we will iterate over that list and create an instance of all possible
actions and store them in a dictionary with the name of the action as the key.

.. code :: python

   actions = {}
   for name, action in possible_actions.iteritems():
       actions[name] = action(application, player, entity)
       
Finally we are going to check if the dictionary has a "Look" action and have
the player move to the target and say the result of the action when it reached
it.

.. code :: python

   if actions.has_key("Look"):
       fifeagent.approach_and_execute(player, entity,
           callback=lambda: 
               player.FifeAgent.instance.say(
                       actions["Look"].execute(), 2000)
                      )

:py:meth:`approach_and_execute() <fife_rpg.components.fifeagent.approach_and_execute()>`
is a fifeagent component function that sets the agent to move to a
location or the position of another agent and then execute a function.
The function to executed is passed in the callback argument. We give it a
lambda function that lets the fife instance of the player "say" the text that
the Look action returns.

If you run the application now you should see another entity. If you right
click on that entity the player should move to this entity and a text should be
displayed when the player reaches the entity.

Here is the complete code for this tutorial:

tutorial_scene.py

.. code-block:: python
   :emphasize-lines: 2,4,6,11-12,14,18-
   
   from fife import fife
   from fife.extensions.pychan.internal import get_manager
   
   from fife_rpg.game_scene import GameSceneListener, GameSceneController
   from fife_rpg.components import fifeagent
   from fife_rpg.actions import ActionManager
   
   class Listener(GameSceneListener):
       
       def mousePressed(self, event):
           application = self.gamecontroller.application
           player = application.world.get_entity("PlayerCharacter")
           if event.getButton() == fife.MouseEvent.LEFT:
               map_point = application.screen_coords_to_map_coords(
                               (event.getX(), event.getY()), "actors"
                               )
               fifeagent.run(player, map_point)
           elif event.getButton() == fife.MouseEvent.RIGHT:
               game_map = application.current_map
               if game_map:
                   scr_point = fife.ScreenPoint(event.getX(), event.getY())
                   actor_instances = game_map.get_instances_at(
                                                       scr_point, 
                                                       game_map.get_layer("actors")
                                                       )
               else:
                   actor_instances = ()
               for actor in actor_instances:
                   identifier = actor.getId()
                   outliner = self.gamecontroller.outliner
                   if identifier in outliner.outline_ignore:
                       continue
                   entity = application.world.get_entity(identifier)
                   possible_actions = ActionManager.get_possible_actions(
                                                                player, 
                                                                entity) 
                   actions = {}
                   for name, action in possible_actions.iteritems():
                       actions[name] = action(application, player, entity)
   
                   if actions.has_key("Look"):
                       fifeagent.approach_and_execute(player, entity,
                           callback=lambda: 
                               player.FifeAgent.instance.say(
                                       actions["Look"].execute(), 2000)
                       )
                               
   class Controller(GameSceneController):
       
       def __init__(self, view, application, outliner=None, listener=None):
           listener = listener or Listener(application.engine, self)
           GameSceneController.__init__(self, view, application, outliner, 
                                        listener)
           application.add_map_switch_callback(self.on_map_switched)
           
       def on_map_switched(self):
           renderer = fife.FloatingTextRenderer.getInstance(
                                           self.application.current_map.camera)
           font = get_manager().getDefaultFont()
           renderer.setFont(font)
           renderer.addActiveLayer(self.application.current_map.get_layer(
                                                                       "actors"))
           renderer.setBackground(100, 255, 100, 165)
           renderer.setBorder(50, 255, 50)
           renderer.setEnabled(True)   

run.py

.. code-block:: python
   :emphasize-lines: 6,14,17,19-21

   from fife_rpg import RPGApplication
   from fife_rpg import GameSceneView
   from fife.extensions.fife_settings import Setting
   from fife_rpg.components import fifeagent
   
   from tutorial_scene import Listener, Controller
   
   settings = Setting(app_name="Tutorial 4", settings_file="settings.xml")
   
   def main():
       app = RPGApplication(settings)
       app.load_components("combined.yaml")
       app.load_behaviours("combined.yaml")
       app.load_actions("combined.yaml")
       app.register_components()
       app.register_behaviours()
       app.register_actions()
       view = GameSceneView(app)
       controller = Controller(view, app)
       controller.listener.is_outlined = True
       controller.outliner.outline_ignore.append("PlayerCharacter")
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
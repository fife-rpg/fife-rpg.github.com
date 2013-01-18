Basics
======

Like FIFE FIFErpg applications need a settings_ file. FIFErpg adds a few
settings to the FIFE ones which are inside the "fife-rpg" module. The only one
that is absolutely required is the "ProjectName" setting, which is of the
type "str". This is the (project) name of your game. For the basic example
this is enough, but you would not be able to use most of the features of
FIFErpg. The other settings will be shown when the features are explained
in this tutorial.

.. _settings : http://wiki.fifengine.net/Engine_Extensions#Engine_Settings

An example of a basic settings file for FIFErpg looks like this:

:download:`settings-dist.xml  <basics/settings-dist.xml>`

.. literalinclude:: basics/settings-dist.xml


Note that it is named "settings-dist.xml" and not "settings.xml". This is a
feature that is already in FIFE itself. If it can't find the "settings.xml"
file it will use a settings-dist.xml file to create a default one. So a
"settings-dist.xml" should only contain settings that work on every system.

As mentioned in the fife wiki the settings file is normally placed in the same
directory as the main Python file.

Also while there is no text being actually shown in the basic example a font is
needed. The tutorial uses the FreeSans font. The version used can obtained from
:download:`here  <basics/fonts/FreeSans.ttf>`. You are free, of course, to use
whatever font you want, but you will need to adjust the settings file to point
to the font. The default settings will look for a "FreeSans.ttf" file inside a
"fonts" directory.

With those things taken care of we can move on to the code.

First we need to import three classes from FIFErpg:

.. code:: python

   from fife_rpg import RPGApplication
   from fife_rpg import GameSceneView
   from fife_rpg import GameSceneController
   
The RPGApplication class is the main class for FIFErpg, it takes care of most
of the automatic discovery and activation features. It also stores the world,
which is like the model in a model-view-controller pattern. Every FIFErpg
application should either use this or have a class that inherits from this.

The GameSceneView and GameSceneController classes are dummy classes that
can be used when no actual functionality is required for either the controller
or the view. For the basic example these will suffice, but you will need to
write your own views and controllers, and inherit those from
ViewBase and ControllerBase respectively. That will be for a later tutorial
though.

Also you will need to import the settings extension from fife:

.. code:: python

   from fife.extensions.fife_settings import Setting
   
This will be used to load the settings file, which is the next step:

.. code:: python

   settings = Setting(app_name="Tutorial 1", settings_file="settings.xml")
   
The next thing is to create the application, using the settings:

.. code:: python

   app = RPGApplication(settings)
   
The view and controller also need to be created. Note the order, it is
important:

.. code :: python

    view = GameSceneView(app)
    controller = GameSceneController(view, app)   

The next step is to have the application create the world:

.. code :: python
   
   app.create_world()

Now everything is set up. But you will need to tell the application to use
the controller. This is done with the following code:

.. code :: python

   app.push_mode(controller)
   
The "push_mode" method is inherited from the bGrease class BaseManager_ and the
FIFErpg controllers are specialized modes.

.. _BaseManager : http://beliaar.github.com/bGrease/mod/mode.html#bGrease.mode.BaseManager.push_mode

Finally, the only thing left is to actually start the application.

.. code :: python

    app.run()

Here is the complete code:

.. code :: python

   from fife_rpg import RPGApplication
   from fife_rpg import GameSceneView
   from fife_rpg import GameSceneController
   from fife.extensions.fife_settings import Setting
   
   settings = Setting(app_name="Tutorial 1")
   
   def main():
       app = RPGApplication(settings)
       view = GameSceneView(app)
       controller = GameSceneController(view, app)
       app.create_world()
       app.push_mode(controller)
       app.run()
       
   if __name__ == '__main__':
       main()
       
If you run this code a window with a black background will appear.
You can either use the Escape key, or the X button in the title bar of the
window to close it.

The :ref:`next <display>` part of the tutorial will show how to actually display a FIFE map.

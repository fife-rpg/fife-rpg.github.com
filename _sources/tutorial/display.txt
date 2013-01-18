.. _display:

Getting something displayed
===========================

The last Tutorial was about the general setup of FIFErpg projects and ended
with getting a black window displayed. Since that is not very interesting this
tutorial will be about loading and displaying maps.

Maps
----
FIFErpg uses the normal FIFE map files for the static non-interactive objects
of a map and thus the first thing needed is an actual map. If you don't have a
map yourself you can download this :download:`archive <display/maps.7z>`. Extract
that archive to the directory where your settings-dist.xml is. It should create
a maps and a objects directory. The latter contains the graphics for the map.
The map itself is in the maps directory. It is the same map that is used in
the RPG demo of FIFE.

New Settings
------------
Before this map can be loaded you need to add settings to your
settings-dist.xml.
The first is a 'Camera' setting. This is the camera that will be used after
loading the map. For the downloaded map this is 'camera1'. Change this if you
use a different map.
The second is the 'Components' setting. This is a list that tells FIFErpg which
components are used by the application. This will be explained more later.
For now the list only needs to contain 'Agent'.

You can verify with the updated
:download:`settings-dist.xml <display/settings-dist.xml>` file.

.. hidden-literalinclude:: display/settings-dist.xml

New Files
---------
There are also new definition files that need to be created.
First you need a file that contains a list of what components are available and
what names they have. That file is a YAML file with a simple structure:

.. code:: yaml

   Components:
      <component_name>: <python path to the component module>
               
Example:

.. code:: yaml

   Components:
      Agent: fife_rpg.components.agent
 
:download:`combined.yaml` contains the standard FIFErpg components, as well as
a list of systems, actions and behaviours which will be explained later.
Also a list of the available map is needed. By default FIFErpg looks for a
'maps.yaml' file inside the 'maps' subdirectory. This is a simple YAML file
with that has the following structure:

.. code:: yaml

   Maps:
      <name of the map>: <filename of the map, without the extension>
      
Example:

.. code:: yaml
   
   Maps:
      Level1: "level1"
      
The above example is also what is needed for this tutorial, but you can download
the file :download:`here <display/maps/maps.yaml>`.

The code
--------
With those changes down we can get to the additions to the code.

The components have to be loaded and activated after creating the application
and before creating the world. After creating the application here:

.. code:: python

   app = RPGApplication(settings)
   
Add the following lines:

.. code:: python

    app.load_components("combined.yaml")
    app.register_components()

:py:meth:`load_components("combined.yaml") <fife_rpg.rpg_application.RPGApplication.load_components>`
loads the components that are in the "combined.yaml" file and adds them to an
internal list.

:py:meth:`register_components() <fife_rpg.rpg_application.RPGApplication.register_components>`
registers the components that are set in the 'Components' setting.

The maps can be loaded and activated anywhere between creating the world and
running the main loop of the application. After the line:

.. code:: python

   app.create_world()
 
Add the following:

.. code:: python

    app.load_maps()
    app.switch_map("Level1")
    
:py:meth:`load_maps() <fife_rpg.rpg_application.RPGApplication.load_maps>`
loads the list of maps from the maps.yaml file.

:py:meth:`switch_map("Level1") <fife_rpg.rpg_application.RPGApplication.switch_map>`
changes the active map to "Level1".

Here is the complete code:

.. code-block:: python
   :emphasize-lines: 10-11, 15-16

   from fife_rpg import RPGApplication
   from fife_rpg import GameSceneView
   from fife_rpg import GameSceneController
   from fife.extensions.fife_settings import Setting
   
   settings = Setting(app_name="Tutorial 2", settings_file="settings.xml")
   
   def main():
       app = RPGApplication(settings)
       app.load_components("combined.yaml")
       app.register_components()
       view = GameSceneView(app)
       controller = GameSceneController(view, app)
       app.create_world()
       app.load_maps()
       app.switch_map("Level1")
       app.push_mode(controller)    
       app.run()
       
   if __name__ == '__main__':
       main()

If you run this code you should see something like this:

.. figure:: display/screenshot.png
   :align:   center
   
   Empty map. This may vary if you used a different map.
   
If it appears that there are graphical problems you might want to check out the `Known
Problems <http://wiki.fifengine.net/Known_Problems>`_ section of FIFE.

In the :ref:`next <entities>` part of the Tutorial we will be adding the Dynamic objects
to the map.
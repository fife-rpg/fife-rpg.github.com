.. _systems:
.. include:: ..\include.inc

Systems
=======
Besides entities and components there is a third part of the component based
entity system: Systems. Basically FIFErpg Systems work the same as
`bGrease Systems`_ . The only addition is that they use the same registering
methods as Components and Actions. The manager can be accessed with:

.. code:: python

   from fife_rpg.systems import SystemManager

The setting name in the settings file is "Systems"

.. _bGrease Systems: http://beliaar.github.io/bGrease/mod/grease.html#bGrease.System

The default systems of FIFErpg are explained in separate tutorials:

.. toctree::
   :maxdepth: 2
   
   scripting
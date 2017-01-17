.. _lighting:

.. include:: ..\include.inc

Lighting
========

This tutorial will explain the Lighting system of FIFE and how it can be
accessed in FIFErpg.

Overview
--------
The Lighting system is used to set and add lights to FIFE maps. We will
first look at the types of light FIFE uses and later how to add to a map with
FIFErpg.

FIFE distinguishes between 3 light types: Simple, (Resized) Image and Animation
and a camera specific lighting color.

Camera lighting color
---------------------
This determines the global lighting used by the camera.

Simple lights
-------------
A simple light is a circular or elliptic light that is created from 4 values:

- Intensity: A value between 0 and 255. Determines the brightness of the light.
  The light will be weaker towards the edges of the circle.
- Radius: The radius of the circle in pixels.
- Subdivisions: How many points the circle will have. More means that the edges
  of the circle are smoother.
- Color: The color of the light as 3 values (RGB) between 0 and 255.
- x- and y-stretch: A float value by which the light should be stretched in the
  x- and y-direction respectively.

(Resized) Image lights (lightmap)
---------------------------------
An image light is a light that is created from a lightmap. The color values of
the lightmap will determine the form, color and brightness of the light. A
resized image light is like a image light with the image being resized to a
specific height and width.

Animation
---------
This is like an image light but with an animated lightmap. There is no resized
version of this.

Common Arguments
----------------
All lights have 3 arguments in common:

- n (RendererNode): This determines the position of the light. A detailed
  explanation is below.
- dst/src: The destination and source blend modes.

RendererNode
------------
The RendererNode determines the position of the light. For lights this means
specifically:

Instance : A fife.instance which the light will follow.

Layer: The layer the light originates from. Lights will
illuminate lower layers, but not higher ones.

Location: The relative or absolute location of the light depending
on whether the Instance was set or not.

point: The relative or absolute window position as a fife.Point.
This differs from location as it is in pixels and (0, 0) is the
upper left position of the window.

Usage in FIFErpg
----------------
For FIFErpg the lighting color for the camera of the current map can be set
by using the
:meth:`~fife_rpg.rpg_application.RPGApplication.set_global_lighting`
method of the |Application|.

The other lights can be added by using methods of the |Map|, specifically
:meth:`~fife_rpg.map.Map.add_simple_light`,
:meth:`~fife_rpg.map.Map.add_light_from_lightmap` and
:meth:`~fife_rpg.map.Map.add_light_from_animation`.

These will take the arguments for the light and also those for the renderer
node. They will also return the LightInfo for the newly created light which can
be used to access and set the information of that light.
.. _overview:

.. include:: include.inc

FIFErpg Overview
================

FIFErpg is a framework for creating games utilizing a component-based entity
system. It uses the FIFEngine and a modified version of the Grease framework.
While the default components are specialized for RPGs it is not limited to
those as one can create their own components and specify what components are
used. The project started as a kind-of fork from PARPG but has been mostly
rewritten to utilize components.

The goals of FIFErpg are:

* To create a framework that makes it easy to create component based games
  with the FIFEngine by taking care of the basic setup needed for FIFE
  applications and providing the component system.
* To create tools that allows to edit the game assets like maps, dialogues,
  and scripts.

Currently the first is (mostly) finished while there are no actual tools.

License
-------
FIFErpg is licensed under the GNU General Public License Version 3.
The license text can be found in the gpl-3.0.txt file in the root directory of
FIFErpg.

Requirements
------------
FIFErpg requires a Python 2.7 installation with the following modules:

* FIFE - http://www.fifengine.net
* bGrease - https://github.com/Beliaar/bGrease
* PyYAML - http://pyyaml.org/

Downloading FIFErpg
-------------------
Currently there are two methods to obtain FIFErpg:

Download an official release
............................

These can be obtained here:

https://sourceforge.net/projects/fife-rpg/files/

Clone the git source repository
...............................

The following command clones the repository:

.. code:: bash

 git clone git://github.com/fife-rpg/fife-rpg.git

Installation from source
------------------------

If you downloaded a source release or cloned the repository you have to run
the following command:

.. code:: bash
 
 python setup.py install

in the root directory of the source.

After that you can test that everything was correctly installed by cloning the
demo repository

.. code:: bash 
 
 git clone git://github.com/fife-rpg/fife-rpg-demo.git

and then running

.. code:: bash
 
 python run.py

inside the root directory of the demo.

If that opens a new window then everything was correctly installed.
If the window is black, or the graphics appear blurry or otherwise incorrect
it is mostly a problem with FIFE as that handles the graphical representation.
You should check the `Known Problems`_ page.

.. _Known Problems: http://wiki.fifengine.net/Known_Problems

Documentation
-------------
You can view the documentation online at http://fife-rpg.github.com/.
This will always contain the documentation for the Git repository.

Development status
------------------

FIFErpg is currently under development and has no official release yet.
The first official release, which will probably have a version number of 0.1,
is planned to be made after the next version of FIFE is released and this
manual is finished, or does have at least a Tutorial.

Contributing
------------
The easiest way to contribute is to make a fork of the Git repository
on Github and create a pull request.

You can check the issues_ page for unassigned tasks. If you work on an issues
check the comments if someone else is working on it and write a comment if you
are working on it.

You can of course create issues yourself, either as proposals for others or
tasks that you work on yourself. Please note that there is no guarantee that
issues that you create yourself are being added to FIFErpg.

.. _issues: https://github.com/fife-rpg/fife-rpg/issues
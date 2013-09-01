.. _charstats:

.. include:: ..\include.inc

Character Statistics
====================

This tutorial will explain the |CharacterStatistics| system of FIFErpg.

Overview
--------
The character statistics system is based on the character statistics system
that was planned for PARPG. It consists of primary and secondary - or derived -
statistics.  The statistics can be raised by paying statistic points. The cost
of raising a statistic depends on the how far away it is from the default value.

There is no default set of statistics and one can freely set what primary and
secondary statistics are available either using the methods of the system or
by writing a YAML file.

Primary and secondary Statistics
--------------------------------
Primary statistics are statistics that can directly be influenced by spending
statistic points to raise them.

Secondary statistics are statistics which are influenced by primary statistics.
A secondary statistic has a list of primary statistics with a float number that
sets how much it influences the statistic. The influences are added to calculate
the value of the secondary statistic. Example:

The Melee Damage(MD) is influenced by the Strength(ST) and the Coordination(CO).

The influences are as follows:

0.7 ST and 0.3 CO. Note that the values do **not** have to add up to 1.0.

ST is 72 and CO is 57. The calculation is as follows:

.. code::

   MD = ST * 0.7 + CO * 0.3 
   MD = 72 * 0.7 + 57 * 0.3
   MD = 50,4 + 17,1
   MD = 67,5

Adding statistics and accessing the values
------------------------------------------
The easiest way to add statistics is to create a yaml file and load it with
:py:func:`load_statistics_from_file()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.load_statistics_from_file()>`

Here is an example:

.. code::

   primary:
     ST:
       name: Strength
       description:
     CO:
       name: Coordination
       description:
   secondary:
     MD:
       name: Melee Damage
       description:
       influences: 
         ST: 0.7
         CO: 0.3

ST, CO and MD are the short forms of the statistic and used for accessing these.

You can also use the
:py:func:`add_primary_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.add_primary_statistic()>`
and
:py:func:`add_secondary_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.add_secondary_statistic()>`
methods to add statistics.

To actually access the values you need to have an |RPGEntity| with a
CharacterStatistics component. A primary statistic is access with the
primary_stats field and a secondary statistic with the secondary_stats field.
Note that you **can** set a secondary statistic but it will be recalculated at
the next pump.

Example:

.. code:: python

    player.CharacterStatistics.primary_stats["ST"] = 72
    player.CharacterStatistics.primary_stats["CO"] = 57
    
    #After the application has performed a pump 
    print player.CharacterStatistics.secondary_stats["MD"]
    #Output: 67.5

Increasing and decreasing statistics
------------------------------------
While it is possible to directly set the values for primary statistics you have
to use the methods from the system class to use the statistic point system.

To get the available statistic points for an antity use the
:py:func:`get_statistic_points()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.get_statistic_points()>`
method. It takes either an |RPGEntity| or the name of an entity.

To get the cost of increasing or the gain of decreasing a statistic use the
:py:func:`get_statistic_increase_cost()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.get_statistic_increase_cost()>`
or
:py:func:`get_statistic_decrease_gain()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.get_statistic_decrease_gain()>`
methods respectively. These take an |RPGEntity| or the name of an entity and the
short name of the statistic.

To check if a statistic can be increased or decreased us the
:py:func:`can_increase_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.can_increase_statistic()>`
or
:py:func:`can_decrease_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.can_decrease_statistic()>`
methods respectively. These take an |RPGEntity| or the name of an entity and the
short name of the statistic.

To increase or decrease a statistic use the
:py:func:`increase_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.increase_statistic()>`
or
:py:func:`decrease_statistic()
<fife_rpg.systems.character_statistics.CharacterStatisticSystem.decrease_statistic()>`
methods respectively. These take an |RPGEntity| or the name of an entity and the
short name of the statistic. They will check if the statistic can be increased
or decreased and adjust the statistic points to the cost or gain.

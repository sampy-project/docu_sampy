# Movements in SamPy

## liste des points





# to reformat later

In this section we present in details how agents' movements are encoded in SamPy. In order to do so, we need to first explain a few notions. If you are only interested in the detailed description of the movements' methods, you can skip to the methods section.

## Required notions

intro

## How are agents' position encoded in SamPy

In samPy's ABMs, time is discretized (with a time-step generally corresponding to a week), and each agent has two different localization attributes, `territory` and `position`, at each timestep `t`. Each of this attribute contains the `index` of a vertice of the graph on which the agents live. Essentially, the `territory` attribute corresponds to the `cell` on which the agent has most of its territory (which stays the same for egenrally more than one timestep), while the `position` attribute corresponds to the cell on which the agent spends its time during a specific timestep. Therefore, those attribute are generally not the same.

## what is a moevement

In SamPy, a movement is a sequence of steps

### what is a step



bla bla bla

## Methods
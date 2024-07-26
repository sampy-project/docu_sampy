# How to add interventions in a model

One of the main motivation for Sampy's development was to have a tool able to run efficiently a large number of simulations in order to assess the efficiency of planned intervention strategies in the context of rabies in North America. As such, Sampy includes a series a built-in intervention object used to represent real life interventions on an epidemics (like vaccination).

**_NOTE:_** the assessment of intervention strategies is a context in which a lot of simulations are required. Therefore, it calls for models running in parallel on multiple processes. Sampy provides a few tools to ease the management of such models, but since multiprocessing in python requires at least some knowledge of python (and is OS dependant), those will be detailed in an advanced guide.

## Choice of the intervention

Currently Sampy provide two built-in interventions, `BasicCulling` and `BasicVaccination`, the first modelling culling intervention (the targeted removal of agents) and the later modelling vaccination (the targeted immunization of agents). Here we will focus on `BasicVaccination` (the culling object behaves roughly the same as the vaccination one).

## How to use the vaccination object

A `BasicVaccination` object works as follows.

1. It targets one specific disease, which is given by the user at construction.
2. Vaccines stay effective for a user defined duration. While under the effect of an effective vaccine, an agent is totally immuned to the disease.
3. Several methods are available to vaccinate the population, and they all work on the same principle: the user provides a locations (i.e. vertices) with associated probability values representing the chance for an agent on those locations to be vaccinated.

To be instanciated, the vaccination object requires a disease object as well as an integer. This second parameters represents the number of timesteps a vaccinated agent remains immunized to the disease.
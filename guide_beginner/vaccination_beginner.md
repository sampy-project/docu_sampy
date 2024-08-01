# How to add interventions in a model

One of the main motivation for Sampy's development was to have a tool able to run efficiently a large number of simulations in order to assess the efficiency of planned intervention strategies in the context of rabies in North America. As such, Sampy includes a series a built-in intervention object used to represent real life interventions on an epidemics (like vaccination).

**_NOTE:_** the assessment of intervention strategies is a context in which a lot of simulations are required. Therefore, it calls for models running in parallel on multiple processes. Sampy provides a few tools to ease the management of such models, but since multiprocessing in python requires at least some knowledge of python (and is OS dependant), those will be detailed in an advanced guide.

## Choice of the intervention

Currently Sampy provide two built-in interventions, `BasicCulling` and `BasicVaccination`, the first modelling culling intervention (the targeted removal of agents) and the later modelling vaccination (the targeted immunization of agents). Here we will focus on `BasicVaccination` (the culling object behaves roughly the same as the vaccination one).

## How to use the vaccination object

A `BasicVaccination` object works as follows.

1. It targets one specific disease, which is given by the user at construction.
2. Vaccines stay effective for a user defined duration. While under the effect of an effective vaccine, an agent is totally immuned to the disease.
3. Several methods are available to vaccinate the population, and they all work on the same principle: the user provides locations (i.e. vertices) with associated probability values representing the chance for an agent on those locations to be vaccinated. 

To be instanciated, the vaccination object requires a disease object as well as an integer. This second parameters represents the number of timesteps a vaccinated agent remains immunized to the disease. The object creation is done as follows.

```python
from sampy.intervention.built_in_interventions import BasicVaccination

# [population and disease creations here]

vaccination = BasicVaccination(disease=disease, duration_vaccine=156)
```

Once created, this object should be used in the main loop of the model as follows.

1. At the beginning of each iteration of the main loop, the method `update_vaccine_status` (does not take any parameters) should be called. Essentially, it performs a series of internal updates required for the vaccine to work properly.
2. There are two main methods to apply vaccine to the population, `apply_vaccine_from_array` and `apply_vaccine_from_dict`. The former is more optimized but requires some understanding of Sampy's internal components, so we will focus on the latter. It expects two argument, the graph object and a dictionnary `dic`, whose keys are id of vertices and values are floats between 0 and 1. Basically, if a `vertex_id` is a key in this dictionnary, then each agent on `vertex_id` at the time of application has a probability of `dic[vertex_id]` to be vaccinated.

In the code, it looks like this.

```python
# start of the main loop itself
nb_year_simu = 5
for week in range(nb_year_simu * 52):

    vaccination.update_vaccine_status()
    if week == 30: # we apply vaccine at week 30 of the first year here.
        vaccination.apply_vaccine_from_dict(my_graph, dic)

    # [rest of the loop]
```

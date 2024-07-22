Sampy is a pure python framework for large scale agent based modelling. Here are the core features of Sampy.

1. It is optimized for very large populations (from 100k to a few million agents), and can easilly run on a personnal computer at those scales.
2. It is fast: at least as fast as Agents.jl when comparable. See the Wolf-Sheep example in the [documentation]. 
3. It is designed to feel like the libraries Numpy/scipy, i.e. all the technical details required for optimization are abstracted away from the user.
4. It is modular and users familiar with python can easilly add functionalities specific to their needs. 

## Installation

The easiest way to install Sampy is to use pip with the following command:

```console
$ pip install sampy-abm
```

Otherwise, you can get the source code directly on GitHub [there].

## Who is Sampy for and what can we do with it?

It depends on how proficient you are with Python.

### If you are relatively new with Python

Sampy comes with several "ready-to-use" objects that you can employ to build your ABMs. We also provide series of already made ABM in which you can plug your own parameters. 
The list of available ready-to-use tools is continuously growing, but note that most that are currently available target ecology and epidemiology of zoonotic diseases applications.

### If you are comfortable with Python

Sampy is highly modular by design: each object is composite and obtained by multiple inheritance of several "building blocks", each one adding new functionalities to the object. You can think of those blocks as mixins, even if they technically are not. Thanks to this, you can pick the building blocks corresponding to your modelling needs and add your own. See the [documentation] for guides and examples on how to do this. 

## What makes Sampy different from other ABM frameworks

The core difference between Sampy and other ABM frameworks lies in the [activation scheme] Sampy is optimized for. That is, the way in which the agents perform their actions. To properly understand the difference, it is best to focus on a simplified example. Assume that we want to create an ABM in which the agents perform 3 consecutive actions: action_A, action_B and action_C. 

### In a traditional ABM framework

Generally, ABM frameworks come with an [event scheduler] which deals with the activation of each agent. The most common scheduler handles things like this:

1. the agents are shuffled;
2. the event scheduler pick the first agent in line and have it perform action_A, then action_B and finally action_C;
3. the event scheduler pick the second one and do the same;
4. so on until all the agents have been "activated".

In general, ABM frameworks provide several event schedulers that the user can chose from, but they often are variations of the one described above.

### In Sampy

Sampy focuses on the following activation scheme:

1. the agents are shuffled (if needed), then they perform action_A one after the other;
2. same thing with action_B;
3. same thing with action_C.

This allows a lot of powerful optimization as well as clearer scripts overall (see example below).

## Example

Here we show a simple example of ABM created with built-in objects of the library. We have a population of mammals living on a square grid with Moore neighborhood (i.e. diagonals are included) with a uniform distribution of ressources (the ammount of ressources is stored on the square grid as the parameters K). Time is discretized and a time step represents a week. At each time step, each agent has a probability to die which depends on: 

1. the number of agents on its cell;
2. the amount of ressources (K) available on its cell;
3. an age dependant probability defined by the user (stored in the array `arr_weekly_mortality`). 

Finally, there are the following yearly events:

1. on week 15 of each year, agents look for a mate on their home cell;
2. on week 22 of each year, the pregnant female agents give birth;
3. on week 40 of each year, the agents disperse to colonize new cells.

We start the simulation with 20 couples (a male and a female) of agents on the central cells of the grid.

```python
import numpy as np
from sampy.agent.builtin_agent import BasicMammal
from sampy.graph.builtin_graph import SquareGridWithDiag

# fix a seed for reproducibility
np.random.seed(1789)

# define a mortality array
arr_weekly_mortality = np.repeat([0.6, .4, .3, .3, .3, .6, .6], 52):\
arr_weekly_mortality = 1 - (1 - arr_weekly_mortality) ** (1. / 52.)

# create the landscape
my_graph = SquareGridWithDiag(shape=(100, 100))
my_graph.create_vertex_attribute('K', 10.)

# create the population object
agents = BasicMammal(graph=my_graph)

# create some agents
agents.add_couples(5, [(50, 50), (51, 50), (50, 51), (51, 51)], 52)

nb_year_simu = 100
for week in range(nb_year_simu * 52 + 1):

    if week % 52 == 0:
        print("Population at year", week // 52, ":", agents.number_agents)

    agents.increase_age()
    agents.kill_too_old(52 * 6 - 1)
    agents.natural_death_orm_methodology(arr_weekly_mortality, arr_weekly_mortality)
    agents.kill_children_whose_mother_is_dead(11)

    if week % 52 == 15:
        agents.find_random_mate_on_position(1., position_attribute='territory')
    if week % 52 == 22:
        agents.create_offsprings_custom_prob(np.array([4, 5, 6, 7, 8, 9]), 
                                             np.array([0.1, 0.2, 0.2, 0.2, 0.2, 0.1]))
    if week % 52 == 40:
        can_move = agents.df_population['age'] > 11
        agents.dispersion_with_varying_nb_of_steps(np.array([1, 2, 3, 4]),
                                                   np.array([.25, .25, .25, .25]),
                                                   condition=can_move)
```

This example only build up a population and print out the population count at the first week of each year.
For more detailed examples exploring other functionalities of Sampy (data extraction, disease transmission, vaccination, etc), see the online [documentation].
# How to build up a population

Now that we have chosen the square grid (`SquareGridWithDiag` of shape 100x100), The next step is to create a population of mammals on it. We begin with a short description of what is and is not covered by this guide.

## what is in this section

1. Explaination on what are population objects in Sampy.
2. The difference between the main built-in classes of agents.
3. How to reproduce the life-cycle of a 'typical' mammal with the provided methods (reproduction, dispersion, etc...).
4. How to extract data from the model during runtime.
5. How to save the population for later use in other models.

## what is NOT in this section

1. How to use the more complex built-in populations that have `SCRW` in their name. Indeed, this acronym stands for 'Spherical Corelated Random Walk', a kind of random walk adapted to very long distance dispersions and migrations. Those will have their own guide and associated publication (WIP).
2. How to add genetics to the population.
3. How to modify the population class to add new functionalities. This is covered in the [advanced guide] (WIP).

## What is a population in Sampy

Sampy is an ABM framework, and as such actions and behaviour are resolved at the agent level. However, contrary to most other frameworks, Sampy do not provide a "beginner friendly" way to act directly on individual agents. Instead, Sampy provides `population` objects which allow users to trigger actions on all (or a selection of) the agents in one line of code. As shown below, this design fits very nicely with the activation scheme Sampy is optimized for (see [README]).

## The two main classes of basic mammals

Sampy provides two main population classes designed to mimic generic territorial mammals, namely `BasicMammal` and `BasicMammalPolygamous`. As their name suggest, the main difference between those two population lies in their mating method: in `BasicMammal` males can mate with only one female per 'season', while in the other a male can have multiple mates. Otherwise, they both provide the same basic tools:

1. various methods to deal with natural mortality (age, density and ressource dependant mortality);
2. various methods to deal with dispersion (the natural tendency for animals to spread and colonize new territories);
3. reproduction related methods (find mates and create offsprings).

In this guide, we will work with `BasicMammal`.

## Life-cycle of our artificial species

Our mammals are designed to look like a simplified version of raccoons in north America. The core differences are that they become adult sooner (11 weeks old), they are more likely to disperse but do not go as far as raccoons can go (this allows for a smoother spread of the population during simulations) and they are less likely to die of natural cause when they are 4-5 years old. These choices ensure that the population can maintain itself at relatively low densities (~10 agents per square). One timestep of the model represents a week and there are 52 weeks per year. The main events happening during each year are as follows.

1. At week 15, agents look for a mate. Females that successfully mate become pregnant.
2. At week 22, pregnant females give birth.
3. At week 40, agents disperse on the grid. Note that, in this simplified example, all agents disperse with the same parameters (no distinction based on age or sex).
4. At the beginning of every week, agents get a week older and some die of natural cause (based on their age, the number of agents and the amount of ressource -`K`- on their home square).

## How to setup the model

In this section we present the model setup. That is, the first part of almost every Sampy project, in which we set the rng seed and create the objects that are used in the ABM. In our current case, we only need two objects: the graph and the population.

```python
import numpy as np
from sampy.agent.builtin_agent import BasicMammal
from sampy.graph.builtin_graph import SquareGridWithDiag

# ------------------
# model setup

# set the rng seed (good practice)
np.random.seed(1789)

# graph creation
my_graph = SquareGridWithDiag(shape=(100, 100))
my_graph.create_vertex_attribute('K', 10.)

# population creation
agents = BasicMammal(graph=my_graph)

# create some agents: 5 male-female pairs on the four central vertices, all 52 weeks old
agents.add_couples(5, [(50, 50), (51, 50), (50, 51), (51, 51)], 52)
```

## The main loop

Now we need to write the model itself, which consists in a for loop in which each iteration represents a new timestep (here, a timestep represents a week). We generally refer to such a loop as the `main loop` of the model.

```python
# ------------------
# main loop

# first we have to define arbitraty parameters for natural mortality:
# we want 60% of the agent to die during their first year, 40% to die during their second year, etc. (see below).
# We turn those yearly percentages into "weekly mortality".
arr_weekly_mortality = np.repeat([.6, .4, .3, .3, .3, .6, .6], 52)
arr_weekly_mortality = 1 - (1 - arr_weekly_mortality) ** (1. / 52.)

# start of the main loop itself
nb_year_simu = 80
for week in range(nb_year_simu * 52):

    agents.increase_age()
    agents.kill_too_old(52 * 6 - 1)
    # natural_death_orm_methodology takes two mortality arrays : one for males, one for females.
    # Here we use the same array for both.
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

## how to extract data from the model

Currently, we have a model that runs but does not output anything, which is not very useful. Sampy provides several ways to extract data from a running model, the two main being:

1. the method `save_population_to_csv` included with most population objects, which allows to save a copy of the population for later use;
2. the method `count_pop_per_vertex` which allows to count how many agents are on each vertices of the graph (this method can be used to count only some agents with the kwarg `condition`). For an example of the use of the kwarg `condition`, we refer the reader to the next part of the guide in which a disease is introduced.

It is worth noting that `count_pop_per_vertex`'s output is not directly usable by user, as it relies on a convention used internaly by Sampy (see the advanced guide). In order to turn it into something more manageable, we can either use the method `convert_1d_array_to_2d_array` of our graph object (this is specific to the square grids and does not exist on other classes of graphs) which converts the count output into 2D arrays of the same shape as the grid, or the function `counts_to_csv` which turns a list of "count array" into a csv where the columns are named using the proper id of the graph vertices (works on every type of graph).

In our example, we will save the population at the very end of our simulation, and we will save the population count per vertices at the first week of every year (as 2D arrays) in order to create an animation showing the population growth. The new main loop with data extraction is as follows.

```python
# start of the main loop itself
nb_year_simu = 80
list_count_pop = []
for week in range(nb_year_simu * 52):

    if week % 52 == 0:
        count_pop = agents.count_pop_per_vertex(position_attribute='territory')
        list_count_pop.append(my_graph.convert_1d_array_to_2d_array(count_pop))

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

agents.save_population_to_csv('./pop_beginner_guide.csv')
```

## Illustration of the population growth

Here we illustrate the population growth with a GIF made from the arrays saved in `list_count_pop`, thanks to the libraries [Matplotlib] and [MoviePy]. It feature 10 frames per second, each frame corresponding to the state of the population at the beginning of a year.

<p align="middle">
  <img src="./assets/beginner_guide_anim_buildup.gif" width="50%" />
</p>

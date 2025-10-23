# Beginner's guide

This it the first guide destined to new users of SamPy. Our aim here is to walk you through the creation of typical SamPy's ABMs using built-in objects, and we do not discuss their implementation details. For a guide on those technical details, please see the [advanced user's guide].

## Description of the ABMs

In this guide, we will create three consecutive ABMs in which the agents are mammal-like animals living on a square grid. 

1. In the first ABM we will start with a few couples of agents in the central square and then let them and the successive generations populate the grid.
2. In the second ABM, we start with a population built from the first ABM and we unleash a disease in the central cell. This disease is somewhat similar to rabies (transmission by direct contact and a very high mortality rate of 80%).
3. The last ABM begins like the second, but we vaccinate a portion of the agents on a disc centered on the origin of the disease outbreak. We will analysis the effect of this strategy depending on the proportion of agents immunized.

## What can be found in this guide

This guide covers the following topics.

1. The two main types of square grids available in SamPy (we do not describe the other possibilities, like hexagonal grids).
2. How to add and change attributes on the grid.
3. How to create a population of agents, how to create new agents and an overview of this population's methods.
4. How to create a disease and spread it.
5. How to create an intervention and how to use it (here the intervention is a Vaccination).
6. How to extract data from the model.

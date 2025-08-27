# Population objects

As explained briefly in the ReadMe, SamPy do not provide individual objects for the agents. Instead, SamPy provides population object which, as their names suggest, represent collections of agents. Typically, in an ecological simulation involving `N` number of species, there will be `N` population objects (one for each species).

## Dataframe df_population and memory layout of agents attributes

Each population object as an attribute called `df_population`, which is a DataFrameXS and is used to store all the attributes of the agents. This is done as follows.

Assume our agents have three attributes called `A`, `B` and `C`, then `df_population` has three columns named, you guessed it, `A`, `B` and `C`. Each row of the dataframe corresponds to exactly one agent in the population, and the attributes of the agents at row `i` are simply `df_population['A'][i]`, `df_population['B'][i]` and `df_population['C'][i]`.

Since our dataframes' columns are numpy arrays, those attributes are stored in memory in contiguous blocks (one for each attribute).

## Difference with usual ABM frameworks and why it increases performances

<p align="middle">
  <img src="./assets/mem_sampy_vs_other.png" width="50%" />
</p>

## Focus on the position of each agent 

The localisation of each agent is especially important in a spatial ABM. 
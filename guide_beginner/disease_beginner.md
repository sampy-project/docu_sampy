# How to model a disease

In the previous part of this guide, we constructed an ABM representing an abstract mammal-like species populating a landscape with constant ressources everywhere. Now, our aim is to start a simulation with a fully grown population, and introduce a disease transmitted by direct contact into the population.

We will work with essentially the same script as the one from the previous part of this guide. The core difference, except for what relates to the disease, is the way we will initially create the population. Instead of using the method `add_couples`, we use the method `load_population_from_csv` to load a population created with the build-up script.

## Choice of the disease

Currently, there are two main classes in Sampy representing diseases transmitted by direct contact, one designed for a single species, and one for two. There are other options to add things like contact tracing (i.e. know which agent transmitted the disease to another), but they are still considered in development and are not necessary for our purpose here. We will then focus on `ContactCustomProbTransitionPermanentImmunity`, the built-in one species disease by direct contact.

## How does the disease work?

The chosen disease works as follows:

1. agents can be in three states, suceptible (can be contaminated), infected (contracted the disease but cannot spread it) and contagious;
2. contagious agents have a user defined probability to contaminate each suceptible agents on their location;
3. infected agents have a user defined probabilities to spend `k` timestep infected before moving to the contagious state (more details below);
4. same thing for contagious agents;
5. at the end of their contagious period, agents have a user defined probability to die and agents who survive become permanently immuned.

The methods defining how many timesteps agents are supposed to stay in each state (infected or contagious) expect parameters to be provided in the form of two arrays, `arr_nb_timesteps` (positive integers) and `arr_prob_timesteps` (floats, sums to 1.), where `arr_prob_timesteps[i]` is the probability for an agent to spend `arr_nb_timesteps[i]` timesteps in the considered state.

## How to use it in the code?

A disease object works the same as any other object from Sampy: we first istanciate it (with some parameters given as kwargs) and then use methods from the obtained object to model the disease's dynamic. The disease object can be imported and instanciated as follows:

```python
from sampy.disease.single_species.builtin_disease import ContactCustomProbTransitionPermanentImmunity

# [...] some code here

disease = ContactCustomProbTransitionPermanentImmunity(host=agents, disease_name='disease')
```

As shown above, a disease with default settings only needs a host (population object) and a name (string) to be instanciated. Now the first thing we need to do is to introduce some cases into the population. The simplest way to do it is to use the method `simplified_contaminate_vertices`, which takes the following arguments.

1. `list_vertices`: list of vertices id where the disease should be introduced.
2. `level`: a float between 0 and 1, which is the propability for each agent on the selected cell to be contaminated.
3. `arr_timesteps`: the 1D array of int `arr_nb_timesteps` described in section "How does the disease work" corresponding to the incubation period.
4. `arr_prob_timesteps`: the 1D array of float described in section "How does the disease work" corresponding to the incubation period.

In code, starting the disease at the four central cell with a level of 25% looks as follows

```python
disease.simplified_contaminate_vertices([(50, 50), (51, 50), (50, 51), (51, 51)], 
                                        0.25, np.array([1, 2, 3, 4]), 
                                        np.array([0.25, 0.25, 0.25, 0.25]))
```

At the current stage, we have a disease introduced in our population, and some agents at the center of the map are incubating for 1 to 4 weeks. However, if we let the script as is and run it, nothing different from the build-up script will happen (except for a bit of a slow down due to some memory being devoted to disease states in the population object). To actually introduce the disease into the simulation, we need to add method calls related to the disease dynamics within the main loop. Those methods are:

1. `tick` which automatically update some internal disease-related timer;
2. `mov_around_territory` which is a method of the population object and whose purpose is to make agents move, with some probability, to neighbouring vertices; without it, the disease cannot spread in the landscape; it expects the argument `proba_remain_on_territory` which is a float between 0 and 1 giving the probability for an agent to stay in its home cell during the timestep (biologically, this is often correlated to the notion of 'area of activity');
3. `simplified_contact_contagion` which deals with transmitting the disease; it expects three arguments:
    * `contact_rate` which is the probability for a contagious agents to contaminate a susceptible agent on the cell it is currently on;
    * the arrays `arr_timesteps` and `arr_prob_timesteps` from `simplified_contact_contagion`;
4. `simplified_transition_between_states` whose purpose is to update the disease related status of the agents (infected to contagious, contagious to immuned or dead); it expects 3 arguments: `prob_death`, a float between 0 and 1 which is the probability of dying at the end of the contagious period, `arr_infectious_period` and `arr_prob_infectious_period` that are two 1D arrays (equivalent to `arr_timesteps` and `arr_prob_timesteps`, but for the duration of the contagious period).

A possible implementation is as follows.

```python
# start of the main loop itself
nb_year_simu = 5
for week in range(nb_year_simu * 52):

    agents.increase_age()
    agents.kill_too_old(52 * 6 - 1)
    agents.natural_death_orm_methodology(arr_weekly_mortality, arr_weekly_mortality)
    agents.kill_children_whose_mother_is_dead(11)

    # disease part
    disease.tick() # required to update disease status. if forgotten, nothing disease related
                   # will happen.
    agents.move_around_territory(0.5, condition=agents.df_population['age'] >= 11)
    disease.simplified_contact_contagion(0.1, np.array([1, 2, 3, 4]), 
                                         np.array([0.25, 0.25, 0.25, 0.25]))
    disease.simplified_transition_between_states()

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

With these modifications, we have a script with a disease being unleashed at the center of the map. Now, we need to extract some epidemiological data from the simulation. There are several options, some built-in the disease object, and we list here the three main one.

1. The most generic way to count agents satisfying some criterion is to use the population method `count_pop_per_vertex` with the kwarg `condition`.


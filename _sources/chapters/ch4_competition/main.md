(sec_eval)=
# Competition

## Evaluation

There are a series of Tests which become progressively more difficult.
Within each Test, there are a series of Levels which increase the difficulty within the parameters of the Test.
The Tests proceed until one of the following stopping criterion are met:
- The percentage of missed deliveries, averaged over all Levels within a Test, exceeds a threshold.
- An overall evaluation time limit of two hours is reached.
- All tests are completed.


## Scenarios

### Generative Models
- *Map.*
  We use a randomly generated environment as the basis for each episode.
  As bodies of water introduce constraints as to where airports may be placed, they can become an important factor in air routing problems.
  For this reason, we generate maps using Perlin noise, which is commonly used in the creation of procedurally generated games {cite}`10.1145/325334.325247`.

- *Airports.*
  Airports are placed uniformly at random within the map.
  A pickup zone is placed on the left side of the map and indicates where cargo are generated, whereas a dropoff zone is defined on the right side of the map indicating where thc cargo must be dropped off.

- *Routes.*
  Routes are generated based on the maximum range of aircraft.
  All routes are bidirectional.
  To limit the degree of the route network only a subset of possible routes are placed.
  Two airplane types are defined, one of which is a slower long-range aircraft with larger cargo carrying capacity, and the other of which is a short-range aircraft with a smaller capacity.
  The long-range aircraft is limited in its ability to land at airports in the small pickup and dropoff zones: routes for the long-range aircraft are only connected to a certain percentage of the airports in the zones.
  Meanwhile, short-range aircraft are only able to travel within the zone and to airports adjacent to the zones (i.e., every short-range route must have one node within a zone).
  In general, this means that cargo cannot be delivered by one airplane: a successful delivery can require that the cargo be loaded on up to three airplanes.

- *Cargo.*
  Cargo are generated in the dropoff zone uniformly at random.
  The delivery deadlines are scaled in relation to the shortest path between it's location and destination, as well as average shortest path distances between all airports (to account for pickup time).

- *Dynamic Events.*
  - Routes are randomly taken offline according to a Poisson process.
    The duration that the route is offline is generated uniformly at random within a given range.
    The agent is able to observe the end time of the event once it starts.
  - New cargo is generated during the episode according to a Poisson process.


### Test Set
We provide a [test scenario set](https://airliftchallenge.com/scenarios/scenarios_test.zip) that you can use to evaluate your solution.
Evaluation against these scenarios is described further in the [Getting Started section of the Starter Kit](https://github.com/airlift-challenge/airlift-starter-kit)

The parameter progression is defined by the following Python code block:
```Python
number_of_tests = 40
number_of_levels = 20 # Number of levels within each test


# *** Fixed parameters
# Amount of time it takes an airplane to refuel and load/unload cargo after landing 
processing_time = 10

# Amount of slack time in the soft delivery deadline
soft_deadline_multiplier = 16

# Amount of slack time in the hard delivery deadline
hard_deadline_multiplier = 24 


# ***Parameters used for generating the test progression
max_working_capacity = 10
starting_cargo_per_airport = 6
starting_airplanes_per_airport = 2
num_airports_increase = 4


# *** Test variations
# For each test, we scale up the "static" complexity of the simulation.
# Specifically, for each test we increase the number of airports by a given percentage.
# Then, the remaining parameters are scaled up in relation to the number of airports.

# Number of airports for test 0
num_airports_test0 = 10

# Number of airports for test k
num_airports_k = math.ceil(num_airports_test_kminus1 * num_airports_increase / 100) 

# How many airplanes the airports can process
working_capacity = math.floor(max_working_capacity + test/number_of_tests * (1-max_working_capacity)) 

# Number of cargo to generate initially 
num_cargo = math.ceil(starting_cargo_per_airport * num_airports) 

# Number of airplanes
num_agents = math.ceil(starting_airplanes_per_airport * num_airports) 

# Number of airports inside the drop off zone
num_drop_off_airports = math.ceil(math.log(num_airports)) 

# Number of airports inside the pick up zone
num_pick_up_airports = num_drop_off_airports 


# ***Level variations
# Within each level, we scale up the "dynamic" complecity of the simulation by increasing the number of new cargo generated
# during the episode and rate at which routes go offline.
# For each parameterization, we generate three instances with different random realizations. 
levelset = level // 3

# Number of cargo to dynamically generate during the episode
num_dynamic_cargo = levelset * 5

# Amount of slack time in the soft delivery deadline
dynamic_cargo_soft_deadline_multiplier = 5 

# Amount of slack time in the hard delivery deadline
dynamic_cargo_hard_deadline_multiplier = 15 

# Rate at which the cargo is dynamically generated (Poisson process parameter)
dynamic_cargo_generation_rate = 1/100 

# Rate at which routes go offline (Poisson process parameter)
malfunction_rate = levelset * 1 / 300 

# Minimum duration for which a route will go offline
min_duration = 10 

# Maximum duration for which a route will go offline
max_duration = 100 
```

Examples of various scenario difficulties are shown below.
The first scenario has no dynamic events (i.e., no routes go offline and no dynamic cargo are generated).
The remaining scenarios include dynamic events and show increasing complexity.

```{figure} Test0_lv0.gif
---
name: fig-test0
align: left
---
Test 0, Level 0
```

```{figure} Test0_lv19.gif
---
name: fig-test0_lv19
align: left
---
Test 0, Level 19
```

```{figure} Test10_lv19.gif
---
name: fig-test10
align: left
---
Test 10, Level 19
```

```{figure} Test20_lv19.gif
---
name: fig-test20
align: left
---
Test 20, Level 19
```

An overall score will be generated for each submission from a hidden scenario set which is similar to the test set.

We also provide [simpler scenarios](https://airliftchallenge.com/scenarios/scenarios_dev.zip) which can be used for debugging your solution.
You may build your own set of evaluation scenarios as well. See [Generating Scenarios section](sec_gen) for more information.


## Scoring

First, we assign an episode score.
We use a large weight $\missedcoeff$ to give missed deliveries the largest penalty, and then give a smaller penalty to lateness using a smaller weight $\latenesscoeff$.
Flight cost is penalized by $\totalflightcostcoeff$.
Then, the objective is to minimize the episode score
```{math}
:label: eqn_episodescore
\episodescore
= \missedcoeff * \missed
+ \latenesscoeff * \scaledlateness
+ \totalflightcostcoeff * \scaledflightcost
```

To determine an overall score for the submission, we find it useful to normalized against baseline scores and produce a score that increases with improved performance.
For each scenario, we calculate episode scores $\randomscore$ and $\baselinescore$ for the Random Agent and "shortest path" baseline algorithm, respectively. 
Then given a episode score for a solution, we calculate a normalized score as
```{math}
\normalizedescore
= \frac{\randomscore - \episodescore}
       {\randomscore - BaselineScore}
```
A solution will receive a normalized score of 0 if it only performs as well as a random agent, and will receive a score of 1 if it performs as well as a simple “shortest path” baseline algorithm.
Scores greater than 1 indicate that the algorithm is exceeding the performance of the baselines.
Note that it is possible to obtain negative normalized scores if the agent performs worse than the random agent.

Finally, An overall score is calculated by summing the normalized score over all tests/levels.
```{math}
\overallscore
= \sum_{i = 1}^\textrm{# of tests} \sum_{\textrm{levels in test } i} \normalizedescore(i)
```


## Time Limits

We impose a time limit on the algorithm for each time step (10 seconds), allowing extra time during the first step for pre-planning (10 minutes).
We will ignore any action for a step where a timeout occurs (proceeding to operate based on the previous action).
Subsequent steps will proceed normally.


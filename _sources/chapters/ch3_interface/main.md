(sec_interface)=
# Simulator Interface

The simulator takes the form of a cooperative multi-agent system {cite}`panait2005cooperative` implemented using the [OpenAI Gym](https://www.gymlibrary.dev/) and [Petting Zoo interfaces](https://www.pettingzoo.ml/).
The environment is advanced by calling the environment step method:
```Python
obs, rewards, dones, infos = env.step(actions)
```

This method receives actions for all agents from the algorithm, and updates the internal state of the environment.
It then returns an observation of the environment (which is customizable by the participants) and a reward.
All variables are `obs`, `rewards`, `dones`, `infos`, and `actions` are dictionaries which are indexed by agent ID.
A list of agent IDs can be obtained from `env.agents`.


## Action Space

The simulator implements incorporates aspects of the OpenAI Gym and Petting Zoo "Parallel Environment" reinforcement learning interfaces.
The action space is defined in {numref}`tab_actionspace`. This space adheres to the OpenAI Gym space standard, and incorporates a custom-defined "List" space.
The actions are designed such that the agent does not need to provide fine-grained control of the airplane, but rather can provide a list of "orders" for the airplane to follow at each airport.
For example, by issuing a single action, the agent can order an airplane to (1) load cargo, (2) unload cargo, and (3) take off for a destination when done processing.
At subsequent steps, the airplane state machine will execute the action automatically, and the agent need not issue a new action until the airplane reaches its next destination (or a change in the environment requires an updated action plan).
As the state machine proceeds, it will automatically update the current action (e.g., clearing the process flag when done processing, or emptying the `cargo_to_load` list once the cargo has started loading).

```{list-table} Action Space
:widths: 25 25 50
:header-rows: 1
:name: tab_actionspace

* - Key
  - Space Type  
    (see [Open AI Gym spaces](https://github.com/openai/gym/blob/master/gym/spaces/space.py))
  - Description
* - `process`
  - `Discrete(2)`
  - Flag indicating if the airplane should process when in the WAITING and READY_FOR_TAKEOFF states:  
    0 = Do not process.  
    1 = Process when working capacity is available at current airport.  
* - `cargo_to_load`
  - `List(Discrete(max_num_cargo), maxsize=max_cargo_on_plane)`
  - Cargo ID’s to load onto plane when processing
* - `cargo_to_unload`
  - `List(Discrete(max_num_cargo), maxsize=max_cargo_on_plane)`
  - Cargo ID’s to unload from plane when processing
* - `destination`
  - `Discrete(max_airports + 1)`
  - Contains ID of an airport where the Airplane will travel to next.
```

## Observation Space

{numref}`tab_obsspace` shows the observation space itself.
The observation contains information specific to an agent, whereas the "globalstate" entry contains information global to all agents (including the observations of all other agents).
Note that the "globalstate" value is the same as that returned by the environment's `state()` method.
{numref}`tab_cargoinfo`, {numref}`tab_planetype`, and {numref}`tab_actionspace` describe subspaces that are used in the observation, and 
We use a custom space named DiGraph which represents a NetworkX directed graph.

```{list-table} Observation Space (Dictionary)
:widths: 25 25 50
:header-rows: 1
:name: tab_obsspace

* - Key
  - Space Type
  - Description
* - `plane_type`
  - `Discrete(num_plane_types)`
  - ID of plane type
* - `current_airport`
  - `Discrete(max_airports+1)`
  - ID of airport where airplane currently is located, or `NOAIRPORT_ID` if `MOVING`   
* - `cargo_onboard`
  - `List(Discrete(max_num_cargo), maxsize=max_cargo_on_airplane)`
  - ID's of cargo that is onboard the airplane
* - `max_weight`
  - `Discrete(10000)`
  - Maximum cargo weight that can be carried by airplane
* - `current_weight`
  - `Discrete(10000)`
  - Curent cargo weight carried by airplane
* - `cargo_at_current_airport`
  - `List(Discrete(max_num_cargo), maxsize=max_cargo_on_airplane)`
  - ID's of cargo that is stored at the current airport.
* - `state`
  - `Discrete(4)`
  - Contains the agent state. See: [list of airplane states](Plane_States) and [state machine definition](airplane_state_machine).
* - `available_routes`
  - `List(Discrete(max_airports+1), maxsize=max_airports)`
  - ID’s of airports that can be reached from the current airport. Routes which are disabled will not be included in this list.
* - `globalstate`
  - State Space
  - The global state of the environment.
* - `next_action`
  - Action Space (see {numref}`tab_actionspace`)
  - Contains the last action issued by the agent. Entries will be modified by the state machine as transitions occur.
```

```{list-table} CargoInfo Space (NamedTuple) used in state space.
:widths: 25 25 50
:header-rows: 1
:name: tab_cargoinfo

* - Key
  - Space Type
  - Description
* - `id`
  - `Discrete(max_num_cargo)`
  - ID of the cargo
* - `location`
  - `Discrete(max_airports+1)`
  - ID of airport where cargo is located, or `NOAIRPORT_ID` if cargo is on an airplane or being loaded/unloaded
* - `destination`
  - `Discrete(max_airports+1)`
  - ID of destination airport where cargo is to be delivered.
* - `weight`
  - `Discrete(10000)`
  - Weight of the cargo
```

```{list-table} PlaneType Space (NamedTuple) used in state space.
:widths: 25 25 50
:header-rows: 1
:name: tab_planetype

* - Attribute
  - Space Type
  - Description
* - `id`
  - `Discrete(max_num_cargo)`
  - ID of the airplane type
* - `max_weight`
  - `Discrete(10000)`
  - Maximum cargo weight that can be carried by airplanes of this type
```

```{list-table} StateSpace (Dictionary) used by in observation space. This contains global state that is common to all agents.
:widths: 25 25 50
:header-rows: 1
:name: tab_globalspace

* - Key
  - Space Type
  - Description
* - `active_cargo`
  - `List(CargoInfo,  maxsize=max_num_cargo)`
  - List of active cargo (cargo not delivered or missed)
* - `event_new_cargo`
  - `List(CargoInfo,  maxsize=max_num_cargo)`
  - List of new cargo which was generated and added to the active cargo list in the last step.
* - `agents`
  - Dictionary of agent observations
  - Contains the observation for each agent
* - `plane_types`
  - `List(PlaneType, maxsize=num_plane_types)`
  - Plane types available in this episode
* - `route_map`
  - `DiGraph(number_of_nodes=max_airports, edge_attributes=['cost', 'time', 'route_available'])`
  - Route map which contains information about airports and routes. This is a NetworkX DiGraph class.
```
                   




## Reward
The reward signal is shown in {numref}`tab_rewards`, and follows directly from the objective function defined in *(4)*.

```{list-table} Rewards given at each step of the simulation.
:widths: 75 25
:header-rows: 1
:name: tab_rewards

* - Condition
  - Reward Value
* - When cargo item $c \in \Cargos$ becomes missed
  - $-\alpha$
* - For each time step where cargo item $c \in \Cargos$ is late
  - $-\beta$
* - Each time step during which an airplane is in flight   
  - $-\gamma$
```

## Info Dictionary
The step method returns an `info` dictionary with an entry for each agent.
The value is a dictionary with the following entries:

```{list-table} Info dictionary.
:widths: 25 25 50
:header-rows: 1
:name: tab_info

* - Key
  - Space Type
  - Description
* - `warnings`
  - `List(str)`
  - List of warnings resulting from the action (for example trying to load cargo that is not at the current airport).
* - `timeout`
  - `boolean`
  - (Only returned by the local evaluator env_step method)
    Indicates whether there was a timeout during this step.
    This will occur when the solution policy takes too long to issue the next action. 
```
     
## Stopping criteria
The episode ends when all cargo is either (1) delivered or (2) missed, and when there is no more cargo to be generated by the dynamic cargo generator.








%```{footbibliography}
%:style: plain
%% :filter: docname in docnames
%```


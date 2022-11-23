Essential API
=================

The functions and classes that are necessary to create a solution and run our environment appropriately are contained on this page.
While we do include the API in its entirety, this goal of this page is to assist anyone that needs to know the bare minimum API to run their solutions

*AirliftEnv*
=================

The Airlift Environment is the main interface between the actions given to agents by your solution and the appropriate events occurring based on that input.

Step
---------
.. autofunction:: airlift.envs.airlift_env.AirliftEnv.step

Reset
---------
.. autofunction:: airlift.envs.airlift_env.AirliftEnv.reset


State
---------
.. autofunction:: airlift.envs.airlift_env.AirliftEnv.state

Observe
---------
.. autofunction:: airlift.envs.airlift_env.AirliftEnv.observe

Action Space
--------------
.. autofunction:: airlift.envs.airlift_env.AirliftEnv.action_space

Example
------------
The following codeblock is an example of an environment being initialized with some parameters. The reset function is called
with a seed, actions being created from an observation, as well as the environment being stepped to the next time-step and a new observation returned::

        env = AirliftEnv(initialization_parameters)
        _done = False
        obs = env.reset(seed=379)
        solution.reset(obs, 462)
        while not _done:
            actions = solution.policies(env.observe(), env.dones)
            obs, rewards, dones, _ = env.step(actions)
            _done = all(dones.values())
            env.render()


*Solutions*
===============

This solutions class does not have to be utilized, but it offers key insight how a solution can be created and agent actions can be generated. The solution class can be extended
with your own solution.


Reset
--------------
.. autofunction:: airlift.solutions.solutions.Solution.reset


Policies
--------------
.. autofunction:: airlift.solutions.solutions.Solution.policies


Name
--------------
.. autofunction:: airlift.solutions.solutions.Solution.name


Get State
--------------
.. autofunction:: airlift.solutions.solutions.Solution.get_state

Example
--------------
This example is taken from our random legal agent. We create a new class that extends the Solution class and use the observation
to generate a policy for the agents::

    class LegalRandomAgent(Solution):
        def __init__(self):
            super().__init__()

        def reset(self, obs, observation_spaces=None, action_spaces=None, seed=None):
            super().reset(obs, observation_spaces, action_spaces, seed)
            self._action_helper = ActionHelper(np_random=self._np_random)

        def policies(self, obs, dones):
            return self._action_helper.sample_legal_actions(observation=obs)



Do Episode
--------------
The solutions module also contains a *doepisode* function that can assist in creating the environment and running the defined solution.

.. autofunction:: airlift.solutions.solutions.doepisode


*Agents*
=================
The Agents class contains all the information pertaining to the individual agents. For example weight capacity, speed, their current states and
as well as the loading and unloading of cargo. Just like the environment-- the agent also has its own step function. This step function is separate from the
environment step function and assists the agent in transitioning appropriately between states.

.. _Plane_States:

Plane States
-----------------
Airplanes can be in one of four states. These are all contained in the PlaneState enumeration located in agents.py.

.. autofunction:: airlift.envs.agents.PlaneState

Also see: :ref:`airplane state machine <airplane_state_machine>`

Load Cargo
---------------
.. autofunction:: airlift.envs.agents.EnvAgent.load_cargo

Unload Cargo
----------------
.. autofunction:: airlift.envs.agents.EnvAgent.unload_cargo

Current Cargo Weight
----------------------
.. autofunction:: airlift.envs.agents.EnvAgent.current_cargo_weight




*Action Helper*
=================

This class will assist you in making actions in the environment.

Load Action
-----------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.load_action

Unload Action
--------------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.unload_action

Process Action
---------------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.process_action

Takeoff Action
---------------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.takeoff_action

Noop Action
---------------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.noop_action

Is Noop Action
----------------
.. autofunction:: airlift.envs.airlift_env.ActionHelper.is_noop_action

Example
-----------------

Using the ActionHelper we can issue actions to the agents::

    from airlift.envs.airlift_env import ActionHelper as ah

    # We create a function that does one single agent action
    def do_single_agent_action(env, agent, action, interimstate, donestate, render):
        env.step({agent: action})
        while env.observe(agent)["state"] != donestate and not env.dones[agent]:
            env.step({agent: None})

    # Load Cargo #0
    do_single_agent_action(env, agent, ah.load_action(0), PlaneState.PROCESSING, PlaneState.READY_FOR_TAKEOFF)

    # Unload Cargo #0
    do_single_agent_action(env, agent, ah.unload_action(0), PlaneState.PROCESSING, PlaneState.READY_FOR_TAKEOFF)



*Observation Helper*
=====================

This is a helper class to assist with utilizing the state and observation. Includes helper functions such as getting available destinations or the shortest path route.

Airplane Idle
----------------
.. autofunction:: airlift.envs.airlift_env.ObservationHelper.is_airplane_idle

Available Destinations
-----------------------
.. autofunction:: airlift.envs.airlift_env.ObservationHelper.available_destinations

Get Lowest Cost Path
------------------------
.. autofunction:: airlift.envs.airlift_env.ObservationHelper.get_lowest_cost_path

Get Active Cargo Info
-----------------------
.. autofunction:: airlift.envs.airlift_env.ObservationHelper.get_active_cargo_info

Get MultiGraph
----------------
.. autofunction:: airlift.envs.airlift_env.ObservationHelper.get_multidigraph

Example
-----------------
Using the ObservationHelper we can get important information that assists us in making the proper agent actions. An example
of using the observation helper to check if an agent is idle and also has available routes. After checking for this we then use the "get_active_cargo_info"
function to get active cargo information for that agent::

    # Import the ObservationHelper
    from airlift.envs.airlift_env import ObservationHelper as oh

    obs = env.observe(a)

    if oh.is_airplane_idle(obs) and oh.available_destinations(state, obs, obs["plane_type"]: # Sometimes available destinations is an empty list.
        available_destinations =  oh.available_destinations(state, env.observe(a), obs["plane_type"]]
        actions[a] = {"process": 1,
                      "cargo_to_load": set(),
                      "cargo_to_unload": set(),
                      "destination": numpy.random.choice(available_destinations)}

    # Get info about a cargo that needs to be picked up
    ca = oh.get_active_cargo_info(state, cargo_to_pick_up)



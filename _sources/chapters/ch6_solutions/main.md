(sec_solutions)=
# Solutions

## Vignette: The need for cooperation

% Baseline solutions serve several purposes.
% Not only can they provide a baseline score and serve as verification that the environment is sufficiently difficult to solve, but they also provide a tool for tuning parameters and serve as a starting point for new participants.
% Our environments can be solved in a myriad of ways, ranging from RL-based approaches to operations research techniques with MILPs and heuristics (as described in Section 3), or possibly a combination of the two.

% Cooperation is important both to share resources (in this case working capacity) and ensure on-time deliveries. 

Due to the complexity present in multi-agent environments, the optimal policy of an agent depends not only on the environment, but on the policies of other agents as well.
It is important that agents are capable of coordinating to find a good solution {cite}`Nowé2012`.
As an illustrative example, consider the scenario shown in {numref}`fig-bottleneck`(a).
Cargo must be picked up from airports 1 and 2 and delivered to airport 3.
An optimal policy may route the cargo between airports 1 and 3 via airport 4 (where the airplane can be refueled).
Likewise, it may route cargo between airports 2 and 3 via airport 0.
Now, suppose that the route between airports 4 and 3 become unavailable for a long duration due to a disruption, as shown in {numref}`fig-bottleneck`(b).
In this case, since airport 4 no longer provides a viable route to the destination, a new policy for airplanes at airport 1 may route them through airport 0.
However, this introduces a bottleneck at Airport 0.
If we instead re-calculate the optimal policy for all planes, we may find a new policy as shown in {numref}`fig-bottleneck`(c).
This policy takes into account the interaction of the planes at airport 0, and opts to re-route the cargo from airport 2 through airport 5 instead of airport 0.

```{figure} BottleneckScenario.svg
---
name: fig-bottleneck
align: left
---
Example of a bottleneck scenario that may be faced by an algorithm.
(a) An optimal policy under normal conditions.
(b) A disruption removes one of the routes, and airplanes from airport 1 are re-routed through airport 0, causing a bottleneck.
(c) An optimal policy may re-route airplanes from airport 1 through airport 5 to eliminate the bottleneck.
```


## Related Work

The movement of cargo is typically formulated as a Pickup and Delivery Problem (PDP) where the goal is to pick up and deliver items, often within a specified time window {cite}`BERBEGLIA20108`.
A number of works consider the problem in the specific context of air cargo delivery (see Section 3.1.3 of {cite}`FENG2015263` for a survey of the fleet planning and flight scheduling literature).
The problem is often formulated as a Mixed Integer Linear Program (MILP), which optimizes over profit or cost by assigning aircraft and routes with which to transport cargo.
Several papers further specialize to airlift planning.
The authors of {cite}`doi:10.1287/opre.50.4.582.2864` consider the problem of intercontinental airlift which incorporates a multitude of constraints relating to such factors as fueling, crew rest, and maintenance.
Of note, the decision variables consist of aggregate quantities of airplanes and cargo. This choice allows the problem to be formulated as a linear program, owing to the fact that the continuous optimal values can be rounded to discrete values with little change in the objective value.
An intratheater airlift setting is considered in {cite}`10945-38129`.
The MILP formulation does not explicitly account for time, but instead optimizes over cargo delivered (weighted by priority) within a given day.
Airlift planning is performed at a finer level of granularity in {cite}`doi:10.1287/trsc.2018.0847`, where a MILP assigns individual airplanes to pick up and deliver specific cargo items with the goal of minimizing a weighted sum of total aircraft uptime and delays.

In most cases, the aforementioned formulations provide a pre-specified set of cargo to be transported that are known at the beginning of the episode.
The dynamic PDP incorporates additional items which may appear for pickup during the course of the episode.
Existing optimization-based solutions typically either re-solve the entire static problem when a new demand occurs (at greater computational expense), or update the original solution using heuristics and local searches {cite}`BERBEGLIA20108`.
Some consideration has been given to this problem in the air cargo literature.
An approach is presented in {cite}`DELGADO2021101939` to efficiently respond to cargo demand fluctuations by applying heuristics in conjunction with classic column generation techniques.
However, we are concerned not just with handling new cargo orders, but also adjusting the policy in response to disruptions to flights, routes, etc.
Dynamic programming has been used for routing time-sensitive cargo while taking into account current and historical information regarding delays and other factors {cite}`AZADIAN2012355`.
However, the scope of this work does not cover all of the scenarios possible under our model.

There is continued interest in applying ML techniques to problems typically addressed by the operations research community.
Promising RL techniques have recently been applied to PDPs {cite}`9352489`, as well as the closely related Vehicle Routing Problem {cite}`Joe_Lau_2020`.
There is hope that RL algorithms may be better able to prevent catastrophic results due to disruptions in large transportation networks {cite}`FlatlandNeuripsSlideslive`.
We expect that the development of a light-weight surrogate environment will facilitate application of these new techniques to the airlift planning problem.



## Baselines
-------------------

We include the following two [baselines](https://github.com/airlift-challenge/airlift/solutions/baselines.py):

1) *Random Agent*.
   This agent performs random actions.
   It ensures that the actions are valid, e.g., by ensuring that it only loads cargo that is stored at the current airport.

2) *Shortest Path Agent*.
   The algorithm assigns a single random cargo item to each airplane, and routes the airplane to pickup and deliver the cargo by following the shortest path.
   If the airplane cannot bring the cargo all the way to its destination, the cargo will be dropped off as close to the destination as possible, where it can be assigned to a different airplane.




%```{footbibliography}
%:style: plain
%```


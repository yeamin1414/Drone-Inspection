# SolarGuard

What if a drone could do more than detect a faulty solar panel?

What if it could decide what to inspect, which action to take, when to intervene, and when to leave the problem for a human technician?

That is the problem this project tackles.

SolarGuard AI is a Reinforcement Learning based decision system for autonomous solar-panel inspection. The environment is modelled as a Markov Decision Process (MDP) where a drone observes the condition of a solar-panel cluster, chooses an action, receives a reward, and transitions to a new state.

The optimal inspection strategy is learned using two classic Dynamic Programming techniques:

Value Iteration
Policy Iteration

The resulting policies are then tested across 1,000 simulated inspection episodes to evaluate how well the learned strategy performs under uncertainty.

The Idea

Traditional inspection:

Drone → Detect fault → Report fault

This project explores:

Drone → Understand condition → Choose action → Evaluate outcome → Adapt next decision

That shift from detection to decision-making is the core idea behind the project.

Reinforcement Learning

The environment is formulated as:

MDP = (S, A, P, R, γ)

States

The drone operates across 8 solar-panel condition states, ranging from normal operation and minor dust to severe hotspots, physical damage, post-clean conditions, and faults awaiting human intervention.

Actions

The drone has 5 possible actions:

Action	Purpose
 Fast Pass	Quick inspection
 Thermal Scan	Detect thermal anomalies
 Hover Diagnose	Perform detailed diagnosis
 Physical Clean	Remove dust contamination
 Log & Skip	Escalate for human intervention
Reward Function

The reward function captures the trade-off between:

Finding and resolving faults

vs.

Wasting time and resources on unnecessary actions

The agent therefore learns that the most aggressive action is not always the best action.

🔄 How the Agent Learns

The environment is stochastic, meaning an action does not always produce the same outcome.

The transition model is represented using:

P[action, current_state, next_state]

The agent uses these transition probabilities to estimate the long-term value of each possible decision.

Value Iteration

The agent calculates the expected value of future states and selects the action with the highest expected return.

Policy Iteration

The agent starts with a policy, evaluates it, improves it, and repeats the process until the policy becomes stable.

Using both approaches provides an additional validation layer rather than relying on a single optimisation method.

Example Learned Behaviour

A few intuitive examples:

NORMAL
   ↓
FAST_PASS

No reason to spend expensive inspection resources on a healthy panel.

HOTSPOT_EARLY
   ↓
THERMAL_SCAN

Investigate the anomaly before it becomes a bigger problem.

HOTSPOT_SEVERE
   ↓
HOVER_DIAGNOSE

Spend additional resources when the potential consequence justifies it.

PHYSICAL_DAMAGE
   ↓
LOG_AND_SKIP

Recognise when the right autonomous decision is to escalate rather than intervene.

This is what makes the problem interesting.

The agent is not simply learning "find faults."

It is learning:

"Given what I know right now, what is the best thing to do next?"

Evaluation

The learned policies are evaluated using 1,000 simulated inspection episodes.

The project compares Value Iteration and Policy Iteration using:

Cumulative reward
Reward distribution
Policy agreement
Convergence behaviour
Computational performance

The notebook also visualises the learned policy and state-transition behaviour.

🛠️ Tech Stack

Python
NumPy
Matplotlib

AI / ML
Reinforcement Learning
Markov Decision Processes
Dynamic Programming
Value Iteration
Policy Iteration
Stochastic transition modelling
Reward engineering
Simulation-based evaluation
Why This Is Interesting

Autonomous inspection is becoming increasingly important in renewable energy, robotics, and industrial asset management.

But detecting an anomaly is only one part of the problem.

A useful autonomous system also needs to answer:

What should I do about it?

This project explores that decision layer using an interpretable RL framework.

The current implementation uses a compact tabular MDP, making the resulting policy easy to inspect and explain. The same idea can later be extended toward larger state spaces, computer vision, battery-aware planning, POMDPs, and Deep RL.

🔮 Next Step

The natural evolution of this project would be:

Drone Camera
     ↓
Computer Vision
     ↓
Fault Detection
     ↓
State Estimation
     ↓
Reinforcement Learning
     ↓
Action Selection
     ↓
Scan / Diagnose / Clean / Escalate

Future versions could incorporate:

Thermal and RGB imagery
Computer Vision based fault detection
Battery-aware planning
Weather and flight constraints
Partial observability
Deep Q-Networks
PPO / Actor-Critic methods
Multi-drone coordination
Digital twins
Human-in-the-loop control
What This Project Shows

This project combines reinforcement learning, mathematical modelling, probabilistic reasoning, optimisation, simulation, and autonomous decision-making around a real-world renewable-energy problem.

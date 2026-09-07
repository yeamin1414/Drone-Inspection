# SolarGuard AI

## From Detecting Faults to Deciding What Happens Next

Most drones find the problem. SolarGuard decides what to do about it.

Autonomous inspection systems are getting better at spotting anomalies. Hotspots, dust accumulation, physical damage: modern sensors and computer vision can flag all of it. But flagging a problem and knowing what to do next are two different things, and that second part is where most systems still hand control back to a human.

SolarGuard closes that gap. The drone does not just observe. It decides.

---

## The Problem No One Talks About

Companies like Percepto, Skydio, and Zeitview are already deploying autonomous inspection fleets across solar, wind, and infrastructure sites. The perception side is largely solved. A computer vision model identifies a hotspot. A thermal sensor confirms the anomaly. A monitoring platform flags the underperforming panel.

And then a human still has to decide what to do about it.

That decision layer is what SolarGuard is built for. The agent observes the current panel condition, weighs five possible actions against their costs and uncertain outcomes, and selects the one that maximises long-term expected reward. Not the next reward. The long-term one.

> **"Given what I know right now, what is the most valuable action to take?"**

---

## How It Thinks

Traditional inspection is a pipeline:

```
Drone  →  Detect Fault  →  Report Fault
```

SolarGuard runs a decision loop:

```
Observe Asset  →  Evaluate Actions  →  Estimate Outcomes  →  Act  →  Repeat
```

The difference is not cosmetic. In a pipeline, every fault triggers the same response. In a decision loop, the right response depends on context: what state the panel is in, what the action costs, what happens downstream if the wrong call is made. The agent has to learn when doing less is better than doing more, and that constraint is what makes the problem genuinely hard.

The environment is modelled as a **Markov Decision Process** across **15 inspection stops per flight**, with a discount factor of **γ = 0.95** to keep future consequences in play.

---

## The Environment

### 8 Panel States

| State | What It Means |
|---|---|
| `NORMAL` | No intervention needed |
| `DUST_MINOR` | Light contamination, minor output loss |
| `DUST_HEAVY` | Heavy contamination, cleaning may restore output |
| `HOTSPOT_EARLY` | Thermal anomaly developing beneath the surface |
| `HOTSPOT_SEVERE` | Component failure risk, urgent |
| `PHYSICAL_DAMAGE` | Beyond drone repair, needs a human crew |
| `POST_CLEAN_OK` | Recently resolved, near-normal output |
| `LOGGED_PENDING` | Flagged for crew, awaiting intervention |

### 5 Actions

| Action | The Tradeoff |
|---|---|
| `FAST_PASS` | Low battery cost, may miss developing faults |
| `THERMAL_SCAN` | Catches heat anomalies reliably, moderate cost |
| `HOVER_DIAGNOSE` | Highest accuracy, highest battery draw |
| `PHYSICAL_CLEAN` | Resolves dust contamination directly |
| `LOG_AND_SKIP` | Escalates to crew, zero battery cost |

Every action is a tradeoff between what the drone learns and what it spends to learn it.

---

## Reward Engineering

Two objectives pull against each other throughout every flight:

**Resolving faults that restore output** versus **spending resources on actions that were not necessary.**

An agent that hovers and runs full diagnostics over every panel burns battery without adding value. One that fast-passes a severe hotspot saves battery in the short term and causes a component failure in the long one. The reward function is designed to make both of those outcomes hurt, so the agent learns to find the boundary between them rather than being told where it is.

---

## Uncertainty Is the Point

The environment is stochastic. The same action in the same state does not always produce the same outcome, because real faults do not behave deterministically.

```
P[action, current_state, next_state]
```

A `FAST_PASS` over a `HOTSPOT_EARLY` cluster carries a 40% probability of that fault progressing to `HOTSPOT_SEVERE`, because the early signs were missed. A `THERMAL_SCAN` over the same cluster routes with high probability to `LOGGED_PENDING`, because the anomaly was caught. The agent has to account for that gap in every decision. Rigid rules cannot do that. A learned policy can.

---

## Two Algorithms, One Validation

The optimal policy is found using two independent Dynamic Programming solvers, both implemented from scratch:

**Value Iteration** works backward across the horizon, applying the Bellman update at every state until the value function converges:

```
Q(s, a) = R(s, a) + γ · Σ P(s'|s, a) · V(s')
V(s) ← max_a Q(s, a)
```

**Policy Iteration** starts with an initial policy, evaluates it fully, improves it greedily, and repeats until nothing changes.

Running both is deliberate. If two independent methods converge to the same policy, the formulation is correct. If they disagree, something is wrong.

---

## What the Agent Learns

None of this is programmed in. It emerges from the reward structure and transition dynamics:

```
NORMAL          →  FAST_PASS        No fault, no reason to spend resources
DUST_MINOR      →  PHYSICAL_CLEAN   Cheap to fix now, expensive if left
HOTSPOT_EARLY   →  THERMAL_SCAN     Gather information before it gets worse
HOTSPOT_SEVERE  →  HOVER_DIAGNOSE   Risk justifies the cost
PHYSICAL_DAMAGE →  LOG_AND_SKIP     Drone cannot fix this, escalate
```

That last one is the most interesting result. The agent learns that recognising the limit of its own capability is sometimes the optimal action. Not every fault is the drone's problem to solve.

---

## Results

Both solvers are evaluated across **1,000 simulated inspection episodes** from uniformly random start states.

| Metric | Value Iteration | Policy Iteration |
|---|---|---|
| Convergence time | 2.38 ms | 13.33 ms |
| Outer iterations | 15 | 3 |
| Average episode reward | 57.13 | 58.02 |
| Reward std dev | 42.94 | 41.17 |
| Policies identical | Yes | Yes |

Both converge to the same policy. The reward standard deviation reflects start-state heterogeneity: a flight starting at `PHYSICAL_DAMAGE` earns far less than one starting at `NORMAL`, which is expected and correct.

---

## The Bigger Picture

SolarGuard is currently the decision layer. A full autonomous system would look like this:

```
              DRONE
                │
    ┌───────────┴───────────┐
    ↓                       ↓
RGB Camera            Thermal Camera
    │                       │
    └───────────┬───────────┘
                ↓
       AI Fault Detection
                ↓
        State Estimation
                ↓
     SolarGuard RL Decision Engine
                ↓
        Optimal Action
    ┌──────┼──────┼──────┐
    ↓      ↓      ↓      ↓
  Scan  Diagnose Clean  Escalate
```

The tabular MDP is the proof of concept. The architecture above is where it goes.

---

## What Comes Next

**Computer Vision Integration:** Replace manual state labels with model-predicted inputs from RGB and thermal imagery.

**Battery-Aware Planning:** Add remaining battery, flight distance, and return-to-base constraints into the state representation.

**Partial Observability:** Move to a POMDP where the drone infers asset condition from imperfect sensor readings rather than known ground truth.

**Deep RL:** Scale to continuous state spaces with DQN, Double DQN, PPO, or Actor-Critic methods.

**Multi-Drone Coordination:** Divide inspection zones across multiple agents coordinating decisions at farm scale.

**Human-in-the-Loop Autonomy:** Introduce confidence thresholds so high-risk decisions go to a human operator before execution.

---

## Project Snapshot

| | |
|---|---|
| **Domain** | Renewable Energy / Autonomous Systems |
| **Approach** | Reinforcement Learning via MDP |
| **States** | 8 |
| **Actions** | 5 |
| **Horizon** | 15 inspection stops |
| **Discount** | γ = 0.95 |
| **Solvers** | Value Iteration + Policy Iteration (from scratch) |
| **Validation** | 1,000 simulated episodes |
| **Stack** | Python, NumPy, Matplotlib |

```bash
pip install numpy matplotlib jupyter
jupyter notebook scenario2_drone_inspection.ipynb
# Or upload to Google Colab and hit Run All
```

---

## Final Thought

SolarGuard started with one question: what should an autonomous drone do after it finds something?

The answer is not always to investigate more deeply. Sometimes it is to scan. Sometimes to clean. Sometimes to diagnose. And sometimes the most intelligent autonomous action is to stop, log the problem, and hand it to a human.

That is the idea: move AI from recognising problems to making decisions about them.

---

**43008 Reinforcement Learning, University of Technology Sydney, Spring 2026.**
MDP formulation, transition matrices, reward design, and both solvers are original and written from scratch.

**Yeamin Ahmed** · AI student @ UTS · [LinkedIn](https://linkedin.com/in/yeaminahmed) · [GitHub](https://github.com/yeaminahmed

# P.A.S.T.A. - Planning Algorithm for Strategic Task Allocation

## Project Overview

The EIS (Exploration and Information System) Project is focused on developing an autonomous decision-making system for coordinated UAV-UGV operations. The system manages state tracking, action planning, and strategic exploration in complex environments.

The core components include:

- A comprehensive state management system tracking vehicle positions, tasks, discoveries, and resources
- A flexible action framework supporting various mission-critical operations for both aerial and ground vehicles
- Advanced search strategies utilizing random forests and heuristic-based exploration algorithms
- Predictive state modeling for optimal decision-making

The project aims to maximize mission effectiveness while handling critical constraints such as battery life, task deadlines, and vehicle-specific capabilities.

## State 
```json
{
	"score": null,
	"timestamp": null,
	// UAV
	"uav": {
		"subzone": null,
		"current_task": {
			"type": null,
			"start_time": null,
			"deadline": null,
			"max_value": null,
			"preemptive": null // bool value
		},
		"aruco": [{
			"id": null,
			"zone": null
			// optional position
		}], // list of aruco object discovered
		"object": [{
			"type": null,
			"zone": null
			// optional position
		}],
		"battery": null,
		"explored_subzones": []
	},
	"ugv": {
		"subzone": null,
		"current_task": {
			"type": null,
			"start_time": null,
			"deadline": null,
			"max_value": null,
			"preemptive": null // bool value
		},
		"aruco": [{
			"id": null,
			"zone": null
			// optional position
		}],
		"object": [{
			"type": null,
			"zone": null
			// optional position
		}],
		"battery": null,
		"explored_subzones": []
	}
}
```
## Tasks

### Constraints

The target is considered completed when it is observed for 2 seconds at least and if it is correctly associated with the zone where it is placed.

### Sources

- Mission manager
- PTZ

### Notes

- Find object task: the reward is obtained only if the UAV orbits around the object, else reward is 0 (**REVIEW THIS**)
- Highest spot landing: completed when lands on the highest spot, turn the engine off and send the picture to the GCS
- Emergency landing and return to base should be executed immediately only if the expected reward is the best one wrt other tasks

## Actions

| Tasks | UAV  | UGV |
| --- | --- | --- |
| **Exploration** | ✅ | ✅ |
| **Orbit around object** | ✅ |  |
| **Stop to look at target/object** | ✅ | ✅ |
| **Follow sequence (target)** | ✅ | ✅ |
| **Highest spot landing** | ✅ |  |
| **Emergency landing (UAV)** | ✅ |  |
| **Return to base (UGV)** |  | ✅ |
| **Continue** | ✅ | ✅ |

- Exploration (go to a specific sub-zone) → provided by the exploration algorithm
    - Some probability to find target/object (define how)
- Orbit around object → when object is found by UGV (non-preemptive)
- Stop to look at target/object
- Follow sequence (target)
- Highest spot landing
- Emergency landing (UAV) [Critical → Deadline = 0] **JUST ONE TRY**
- Return to base (UGV) [Critical → Deadline = 0] **JUST ONE TRY**
- Continue → Do nothing and just continue with the current action

## Search strategy

### Notes

Random forest by building more trees using a stochastic approach and then using the highest value solution.

## State prediction

Deterministic.

Each action has its own function to predict the next state from the current one, and takes a certain amount of time.

For each action varies both battery and timestamp.

### Exploration (or go to a sub-zone)

The score is not strictly equivalent to the on of the competition.

The less you the more the exploration gives a better score.

**Status updates**

- `timestamp`: depends on the travel speed and distance with some standard deviation (gaussian)
- `battery`: depends on the travel speed integrated over time with some standard deviation (gaussian)
- `explored_subzones`: to be defined

The target exploration gives an higher scores.

**Targeted exploration strategy** 

For each object and target, takes the possible zone and accumulates them into an heatmap and takes the hottest zone.
If the heap is the same, it returns all of them.

**Exploration for coverage**

To be defined in order to maximize the coverage of unvisited zones following an heuristic.

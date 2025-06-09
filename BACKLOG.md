# Backlog

This document outlines potential areas for expanding the simulation. Each feature lists some implementation suggestions and tests that could guide development.

## Planned Improvements


### Traffic network of connected intersections
- Model downstream effects of one intersection on another
- Shared timing plans or offset coordination between lights
- **Implementation notes**
  - Add an `IntersectionNetwork` class to manage multiple `Intersection` instances and the movement of vehicles between them.
  - Allow network topology to be described via a configuration file (JSON/YAML) to keep setups reproducible.
  - Include travel time between intersections so congestion can propagate realistically.
- **Tests to consider**
  - Check that a vehicle leaving one intersection is eventually queued at the next according to the network layout.
  - Verify that coordinated timing offsets create the expected progression of green waves.

### Adaptive signal control
- Adjust phase order and duration based on real-time traffic conditions
- Could use queue lengths or vehicle counts as inputs
- **Implementation notes**
  - Implement a `SignalController` interface that allows plugging in different control strategies.
  - Provide an adaptive controller that reads lane queue lengths each cycle and adjusts upcoming phases.
  - Keep algorithms modular so new strategies can be added without changing the simulation core.
- **Tests to consider**
  - Force a long queue on one approach and confirm that the adaptive controller extends its green time.
  - Compare average wait time between fixed-time and adaptive control in a small simulation run.

### Realistic traffic flow modeling
- Add dedicated lane objects to represent left, through, and right movements
- Allow lanes to be configured with individual capacities
- Support different arrival distributions for generating vehicles
- Apply turning probabilities so vehicles select lanes realistically
- Model acceleration and deceleration when clearing queued vehicles
- **Implementation notes**
  - Create a `Lane` class with `capacity` and `movement` attributes
  - Update `Intersection` to maintain multiple lanes per approach
  - Provide an `ArrivalDistribution` interface with Poisson and uniform implementations
  - Extend the vehicle generator to choose lanes based on configured turn probabilities
  - Incorporate acceleration delays into the queue discharge logic
- **Tests to consider**
  - Ensure lane queues never exceed their configured capacity
  - Check that arrival distributions produce expected counts over many runs
  - Verify that the percentage of turning vehicles matches the configuration
  - Confirm that enabling acceleration delays increases total clearing time

### Optimize signal timing
- Record queue lengths and waiting times for each direction
- Compute total delay from recorded metrics
- Search for phase durations that minimize the delay
- Provide an offline script to apply the optimizer to saved runs
- **Implementation notes**
  - Extend `metrics.py` to log per-lane queues at every timestep
  - Implement a `calculate_delay` helper summing all waiting time
  - Use a simple hill-climbing search to tweak green durations within bounds
  - Expose a `run_optimizer(data, plan)` function for offline analysis
- **Tests to consider**
  - Verify that optimization reduces total delay on a deterministic scenario
  - Ensure optimized timings respect minimum and maximum duration limits

### Pedestrian crossing logic
- Dedicated crosswalk phases with push-button or timed activation
- Track pedestrian wait times along with vehicles
- **Implementation notes**
  - Add `PedestrianSignal` objects and a method for handling push-button requests.
  - Ensure pedestrian phases do not conflict with vehicle movements when inserted into the schedule.
  - Record wait times for pedestrians in the metrics subsystem.
- **Tests to consider**
  - Simulate a button press and confirm that a pedestrian phase is scheduled within a reasonable time.
  - Check that vehicle and pedestrian phases never overlap in conflicting directions.

### Emergency vehicle priority
- Detect priority vehicles and temporarily override normal phases
- Useful for simulating clearance for ambulances or fire trucks
- **Implementation notes**
  - Mark vehicles with a priority flag and add detection logic to lanes or intersections.
  - When a priority vehicle is detected, preempt the current phase after a short all-red period, then resume normal operation once clear.
  - Keep a record of preemption events to study their impact on overall delay.
- **Tests to consider**
  - Validate that an emergency vehicle clears the intersection faster than regular traffic under identical conditions.
  - Ensure the signal schedule resumes correctly after the priority request is served.

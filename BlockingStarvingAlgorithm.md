# Blocking or Starving Algorithm

There are the following kinds of nodes.

Nodes
- Tank
- Conveyor
- Converter
- Merge
- Split
- Valve

Each node has its own set of rules for how flows pass through the node. What is always true is that flow is conserved, similar to a mass balance in Chemical Engineering.

## Flow Balance Rules by Node Type

### Tank

$\sum In - \sum Out = Accumulation$

If $Accumulation = Capacity$ then

$\sum In \leq \sum Out$

If $Accumulation = 0$ then

$\sum In \geq \sum Out$

### Valve

$\sum In = \sum Out$

$\sum Out \leq MaxFlow$

### Converter

$ConversionFactor \times \sum In = \sum Out$

### Split

$\sum In = \sum Out$

// TODO: Notes on different Split rules

### Merge

$\sum In = \sum Out$

// TODO: Notes on different Merge rules

## The Starved/Blocked Algorithm

The way the Blocked/Starved Algorithm works is by sending a Disruption message through the network so that nodes can determine whether they are in a Blocked or Starved state and which node in the network is causing the problem. It is easy to determine whether an individual node is below capacity without regard to the rest of the network. You simply check whether the flow through the node is at the maximum possible rate. What is more difficult to determine is which node in the network is causing the issue.

Only Valves and Conveyors can be in a Blocked or Starved state. Converters are transparent to signals so signals pass through. Splits and Merges have different rules about passing messages through based on how they are currently configured.

Tanks cannot be Blocked or Starved but they can emit a Disruption signal and in some cases allow a Disruption signal to pass through them.

### Disruption Signal

A Disruption signal is emitted by a node when it is restricting the flow through the network. The signal will include the source node so that reporting metrics can indicate which node is causing the issue. It is entirely possible for a node to be disrupted due to a node that is several locations away in the network. We want to be able to answer these questions.

1. Which nodes are starved or blocked?
2. Which nodes are causing the issue?

### Signal Sources

Any time a node is restricting flow in the network it will emit a disruption signal. The direction of the signal is what determines if the impact is Blocking or Starving. If a node is restricting the flow of nodes connected to the output, then it is considered to be Starving the output nodes. If the node is restricting the input flow rate, then it is considered to be Blocking the input nodes.

1. When a Valve is at its maximum flow rate, it will signal the Input and Output nodes
2. When a Tank is full and has no Output links, it will signal the Input nodes
3. When a Tank is empty and has no Input links, it will signal the Output nodes
4. When a Conveyor is at its maximum velocity, it will signal the Output nodes
5. When a Merge is set to priority order, it will send a signal to each Input whose flow rate is either at its Max Allowed Flow Rate or at a 0.
6. When a Split is set to priority order, it will send a signal to each Output whose flow rate is either at its Max Allowed Flow Rate or at a 0.

## Processing Signals

Each node in a Network will process signals differently based on their current conditions. The following section details how each node processes signals and updates their current state.

### Converters

A Converter is transparent to signals. When it receives a signal, it simply passes it through.

### Tanks

When a Tank receives a signal from its Outputs and the level is currently at the max capacity, it will pass the Blocked signal through to its Inputs.

When a Tank receives a signal from its Inputs and the level is currently at 0, then it will pass the signal through to its Outputs.

In all other cases, the Tank ignores the signal and does not pass it along.

### Valves

If the flow through a Valve is 0 and the signal comes from an Input, then the Valve's state is set to 'Fully Starved' and the signal is passed to the Outputs.

If the flow through a Valve is 0 and the signal comes from an Output, then the Valve's state is set to 'Fully Blocked' and the signal is passed to the Inputs.

If the flow through the Valve is between 0 and the Max Flow Rate and the signal comes from an Input, then the Valve's state is set to 'Starved' and signal is passed to the Outputs.

If the flow through the Valve is between 0 and the Max Flow Rate and the signal comes from an Output, then the Valve's state is set to 'Blocked' and the signal is passed through.

In all other cases, the Valve ignores the signal and does not pass it along.

### Conveyor

When a Conveyor's velocity is at 0 and a signal is received from an Output, the Conveyor's state is set to 'Fully Blocked' and the signal is passed to the Inputs.

When a Conveyor's velocity is between 0 and the Max Velocity and a signal is received from an Output, the Conveyor's state is set to 'Blocked'. The signal is **not** passed to the Inputs.

In all other cases, the Conveyor ignores the signal and does not pass it along.

### Merge

If a Merge is configured to send only one Input, any signal is passed through.

If a Merge is configured to use Percentage or Proportional based rules, any signal is passed to all other connections.

When a Merge is configured to be in priority order and the signal comes from the Input whose flow rate is between 0 and the Max Allowed Flow Rate, the signal is passed to the Outputs.

When the Merge is configured to be in priority order and the signal comes from an Input whose flow rate is either 0 or at the Max Allowed Flow Rate, then the signal is ignored.

When a Merge is configured to be in priority order and the signal comes from an Output, then the signal is passed to the Input whose flow rate is between 0 and the Max Allowed Flow Rate.

### Split

If a Split is configured to send only one Output, any signal is passed through.

If a Split is configured to use Percentage or Proportional based rules, any signal is passed to all other connections.

When a Split is configured to be in priority order and the signal comes from an Input, the signal is passed to the Output whose flow rate is between 0 and the Max Allowed Flow Rate.

When a Split is configured to be in priority order and the signal comes from the Output whose flow rate is between 0 and the Max Allowed Flow Rate, the signal is passed to the Inputs.

When a Split is configured to be in priority order and the signal comes from an Output whose flow rate is 0 or at the Max Allowed Flow Rate, the signal is ignored.



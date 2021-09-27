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

The way the Blocked/Starved Algorithm works is by sending messages through the network so that nodes can determine whether they are in a Blocked or Starved state and which node in the network is causing the problem. It is easy to determine whether a individual node is Blocked or Starved without regard to the rest of the network. You simply check whether the flow through the node is at the maximum possible rate. What is more difficult to determine is which node in the network is causing the issue.

Only Valves and Conveyors can be in a Blocked or Starved state. Converters, Splits, and Merges are transparent for the purposes of being Blocked or Starved. A Blocking or Starving signal will passthrough them. What is important to consider in the case of Merges and Splits though is that they can be configured to only be receiving or sending to a single input or output. Therefore, the configuration of the Merge or Split must be taken into consideration when determining which nodes to pass a Blocking or Starving signal through.

### Sources of Blocking Signal

The following conditions will trigger the emission of a 'Blocking' signal.

1. When a Valve is at its maximum flow rate, its Inputs will receive a 'Blocking' signal
2. When a Tank is full and has no Output links, its Inputs will receive a 'Blocking' signal

### Sources of Starving Signal

The following conditions will trigger the emission of a 'Starving' signal.

1. When a Valve is at its maximum flow rate, its Outputs will receive a 'Starving' signal
2. When a Tank is empty and has no Input links, its Outputs will receive a 'Starving' signal



The only nodes with flow rates limits are Valves and Conveyors. Valves have a Max Flow Rate attribute which restricts how much can flow through the node. Conveyors have a limit on how fast they can move and therefore their maximum output rate is a function of the maximum conveyor velocity and the amount of material loaded onto the conveyor.

When the flow rate going through a Valve is the Max Flow Rate of the Valve then a 'Starved' signal is sent to the Output links and a 'Blocked' is sent to the Input links.

When a Conveyor is at its Max Velocity, then it will send a 'Starved' signal to its Outputs. If a Conveyor's Input is every 0, then it considers itself starved but it does not necessarily send a 'Starved' signal to the outputs. If a Conveyor is 

Some nodes merely pass signals along. Conversion, Merge, and Split nodes will pass the Blocked/Starved signal through since they cannot be Starved/Blocked.

A Tank is a terminating node for Blocked/Starved signals. Tanks are never in a Blocked or Starved state but they can cause other nodes to be in a Blocked or Starved state.
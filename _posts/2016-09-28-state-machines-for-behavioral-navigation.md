---
title: Task Level Control of Robots using Sequence of Behaviors
date: 2016-09-28
permalink: /posts/control-using-behaviors
tags: 
    - motion-control 
    - navigation
    - mobile-robot
---
In dynamic environments, task-level control of robots, where we sequence entire behaviors rather than single action, is important. When we have to navigate an unstructured enviroment, then we cannot use simple navigation techniques using maps. We must be able to react to the environmental conditions. Thus, we need to divide the task of navigation in simple behaviors like *goal_seeking*, *obstacle_avoidance*, *follow_wall*, *go_to_charge* etc. But a fully autonomous robot will be expected to select its own
actions from a larger repertoire of behaviors depending upon the task at hand and the
current conditions.

In my [previous post](/blog/behavioral-control-of-holonomic-robot/), we developed two simple behaviors *go_to_goal* and *obstacle_avoidance* for holonomic robot. But these behaviors can be easily extended to other robots.Goal­ seeking behavior and Obstacle­ avoidance behavior must be combined using some arbitration mechanism to drive the robot to goal while avoiding obstacles. Figure 1 shows motion vectors obtained by using pure behaviors and blended emergent behavior.
{% include image.html url="/images/posts/arbitration.jpg" description="Fig 1. Motion Vectors obtained by various behavior coordination mechanism" %} 

### Hard Switching ###

In hard switching, *go­_to_­goal* and *obstacle_­avoidance* controllers are switched separately with one controller operation at a time. Control operation is switched from *go_­to_­goal* to *obstacle­_avoidance* near any obstacles and switched back to *goal­_to_­goal* when obstacle is cleared. Hard switching guarantees performance of individual controller but may cause unnecessary switching and oscillations between controllers with end effect bumpy motion of robot. Switching between *go­_to_­goal* and *obstacle_­avoidance* controller is shown in Figure 2. 
{% include image.html url="/images/posts/hard_switch.png" description="Fig 2. Hard Switching Approach" %} 

### Blending for Emergent Behavior ###

Unnecessary oscillations and chattering between controllers due to hard ­switching can be 
avoided by using blended controller between *go_­to_­goal* and *avoid_­obstacle*. *Go_­to_­goal* and *obstacle_­avoidance* motion vectors are combined in one controller to form a blended controller. Blended controller provides smooth motion however does not guarantee performance of individual controller. Arbitration mechanism using blended controller is illustrated in Figure 3. The blending function ​$$ \sigma(d_o)\ ε\ [0, 1] $$ depends on value of d​o which is distance to nearest obstacle. The blended controller motion vector is obtained as

$$ U_B = \sigma(d_o)U_{GTG} + (1 − \sigma(d_o))U_{AO} $$ 

where the value of $$ \sigma​(d_o) $$ corresponds the relative weightage ​$$ U_{GTG} $$ with respect to ​$$ U_{AO} $$. Higher value of $$ \sigma​(d_o) $$ means higher weightage to ​$$ U_{GTG} $$ than ​$$ U_{AO} $$ and vice­versa. The value of blending function ​$$ \sigma​(d_o) $$ depends on ​$$ d_o $$ and its value is directly proportional to ​minimum distance to obstacle ($$ d_{min} $$) implying obstacle avoidance vector ​UAO gets most emphasis when distance to nearest obstacle is minimum.
{% include image.html url="/images/posts/blended_controller.png" description="Fig 3. Blended Behavior Approach" %}

### State Machines for Behavior Switching ###

The basic idea of state machines is that your robot can be in one of a finite number of states, such as “waiting,” “moving,” and “recharging,” each of which maps to a behavior. When one state ends, the system immediately moves into another state (for example, changing from “waiting” to “moving” as the robot starts to navigate around the world). The robot must always be in one and only one of these states, and there must be a finite number of them. Which state the robot transitions to can depend on the outcome of the just-finished state. For example, if the robot just moved to its charging station, it might transition from “moving” to “charging,” rather than to “waiting.” Once it is charged, then might transition from “charging” to “waiting.” The behaviors that correspond to the states “waiting,” “moving,” and “charging” can be encapsulated in the states, and the transitions between them are specified by the structure of the state machine.
{% include image.html url="/images/posts/fsm.jpg" description="Fig 3. Navigation by Behavioral Switching using State Machines" %}
Here, the benefit of both hard­switching and blending approach can be incorporated by using blended controller as intermediary controller and switching between *go_­to_­goal*, 
*obstacle­_avoidance* and blended controller using supervisory controller in top level of the control architecture. The complete navigation strategy using state machines is shown in fig 4.

The robot starts in a state of going towards the goal and *go_to_goal* behavior is started, after it receives goal position. IR range finder senses distance 
to nearby obstacles and if the distance to any obstacle is less than distance threshold i.e. ​$$ d_T $$ and greater than unsafe distance ​$$ d_o $$, then blended controller, combination of *obstacle–avoidance* and *go­_to_­goal* controllers, is implemented. Controller switches back to  *go_­to_­goal* if distance ​$$ d>d_T+ε $$ where ​ε is guard band used to prevent unnecessary switching between controllers which results in oscillations. If the robot is at unsafe distance from obstacle i.e. $$ ​d<=d_o $$, obstacle­ avoidance controller is invoked and when it reaches safe distance controller is switched to blended. If distance to goal ​$$ D_{rg} $$ is smaller than tolerable error ε, then it is assumed to have completed the task and robot stops as shown in Figure 4.

Holonomic robot navigating the dynamic environment using state machines is illustated in the video

<a href="http://www.youtube.com/watch?feature=player_embedded&v=L2yUD7iAUaE
" target="_blank"><img src="http://img.youtube.com/vi/L2yUD7iAUaE/0.jpg" 
alt="complete navigation system" margin="auto" margin-left="100px" width="480" height="360" border="10" /></a>
---
layout: post
title: Goal-Seeking and Obstacle Avoidance Behaviors for Mobile Robot
subtitle: ''
meta-description: 'Goal­seeking and obstacle avoidance behaviors are implemented for 						holonomic robot so that it can react to  the changing 
					environment and provide real­time response to the dynamic environment.'
share-img : /img/goal_seeking.png
comments: true
tag: [Motion-Control, Navigation, Holonomic Robot, PID, Mobile-Robot]
---
The basic idea of behavior based control is to subdivide the navigation task into small, 
easy to manage, program and debug behaviors. The world is fundamentally unknown and 
changing so rather than implementing a fixed plan, we focused on developing a library of 
useful controllers (robot behaviors) and switch among the controllers in response to 
environmental changes. Simple behaviors such as goal­seeking and obstacle avoidance are 
defined and output of these behaviors are blended together in accordance to some 
coordination rule.

The mechanical design of holonomic (3-wheel omnidirectional) robot and inverse and forward kinematics equations for Holonomic Robot used in the algorithm are explained in my previous post [(Kinematic Analysis of Holonomic Robot)](/blog/kinematic-analysis-of-holonomic-robot/). Also, PID control of individual wheel motors can be found in [DC Motor Speed Control using Encoder Feedback](/blog/dc-motor-speed-control/). We can use these behaviors for other robots using akerman drive or bicycle drive, we just have to update the kinematics equations.

### Goal Seeking Behavior ###
Goal­seeking behavior implements dead­reckoning based on odometry information and is  
responsible for movement of robot base from initial position to the goal location. The  
robots current location is represented as $$ ξ\ =\ (​x\ y\ θ)^T​ $$  and goal location be $$ G\ =\ (​x_g\ ​y_g\ θ_g)^T $$​.  
{% include image.html url="/images/posts/goal_seeking.png" description="Fig 1. Goal Seeking Behavior Implementation" %} 
This behavior drives the robot towards goal location depending on the two input variables: the distance between the robot and the goal, $$ ​D_{rg} $$ and heading direction towards the goal, $$ \phi_d $$. 

---
**Algorithm: Goal-Seeking Behavior**
$$
\begin{align*}
& {\bf Inputs:}\ D_{rg},\ \phi_d,\ tolerable\_error,\ P(constant\ that\ determines\ speed\ of\ robot\ to\ goal)\\
& Obtain\ the\ goal\ location\ G\ = \
		\begin{pmatrix}
        	x_g\\
        	y_g\\
			θ_g
		\end{pmatrix}
	and\ current\ position\ ξ\ = 
		\begin{pmatrix}
        	x\\
        	y\\
			θ
			\end{pmatrix} 
	with\ respect\ to\ the\ world\ coordinates.\\
& {\bf while}\ Forever\ {\bf do}\\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ euclidean\ distance\ from\ robot\ to\ goal\ D_{rg} = \sqrt{(x_g-x)^2+(y_g-y)^2}\\
& \ \ \ \ \ \ \ \ \ \ \
{\bf if}\ D_{rg} < tolerable\_error\ and\ goal\ is\ reached\ {\bf then}\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ {\bf exit}\\
& \ \ \ \ \ \ \ \ \ \ \ {\bf end\ if}\\
& \ \ \ \ \ \ \ \ \ \ \
Desired\ heading\ direction\ \phi_d\ =\ arctan \left(\frac{x_g-x}{y_g-y}\right)\\
& \ \ \ \ \ \ \ \ \ \ \
K\ = P * D_{rg},\ where\ K\ is\ a\ constant.\\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ desired\ x,\ y\ and\ angular\ velocity\ as\ \dot{x} = K*\cos(\phi_d),\ \dot{y} = K*\sin(\phi_d)\ and\ \dot{θ} = K*(\theta_g − \theta).\\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ individual\ wheel\ velocities\ using\ inverse\ kinematics\ as\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 	
	\begin{bmatrix}
        V_1\\
        V_2\\
		V_3
	\end{bmatrix}
	=
	\begin{bmatrix}
        -\cos(\theta) & -\sin(\theta) & L\\
        \cos\left(\frac{\pi}{3}-\theta \right) & \sin\left(\frac{\pi}{3}-\theta \right) & L\\
		\cos\left(\frac{\pi}{3}+\theta \right) & \sin\left(\frac{\pi}{3}+\theta \right) & L
	\end{bmatrix}
	\begin{bmatrix}
        \dot{x}\\
        \dot{y}\\
		\dot{\theta}
	\end{bmatrix}\\
& \ \ \ \ \ \ \ \ \ \ \
Wheel\ velocities\ are\ maintained\ using\ encoder\ feedback\ and\ PID\ Controller,\ however\ some\ error\ exist.\\
& \ \ \ \ \ \ \ \ \ \ \
Update\ robot\ speed\ using\ actually\ measured\ individual\ wheel\ velocities\ (V_1',\ V_2',\ V_3')\ as\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 
	\begin{bmatrix}
        \dot{x'}\\
        \dot{y'}\\
		\dot{\theta'}
	\end{bmatrix}
	=
	\begin{bmatrix}
        -\frac{2}{3}\cos(\theta) & \frac{2}{3}\cos\left(\frac{\pi}{3}-\theta \right)& \frac{2}{3}\cos\left(\frac{\pi}{3}+\theta \right)\\
         \frac{-2}{3}\sin(\theta) & \frac{-2}{3}\sin\left(\frac{\pi}{3}-\theta \right) & \frac{2}{3}\sin\left(\frac{\pi}{3}+\theta \right)\\
		 \frac{1}{3L} & \frac{1}{3L} & \frac{1}{3L}
	\end{bmatrix}
	\begin{bmatrix}
        V_1'\\
        V_2'\\
		V_3'
	\end{bmatrix}\\
& \ \ \ \ \ \ \ \ \ \ \
Update\ robot\ position\ x=x+\dot{x'}*\delta t,\ y=y+\dot{y'}*\delta t\ and\ \theta = \theta + \dot{\theta'}*\delta t.\\
& \ \ \ \ \ \ \ \ \ \ \
\theta = \theta\ mod\ 2\pi\ as\ \theta\ \varepsilon\ [0,2\pi]\\
& {\bf end\ while}
\end{align*}\\
$$

---

### Obstacle Avoidance Behavior ###
For obstacle avoidance behavior, the robot needs to acquire information about the 
environment and move in such a way as to avoid collision objects that happens to be in 
vicinity of the robot. Obstacles are detected using Sharp IR range Finder which provides 
analog voltage inversely proportional to distance to the obstacle. Eight IR range finder were mounted on top of robot 45 degrees apart.

The distance measured by IR Range Finders in the direction of obstacle is smaller than the distance in other directions. If we take the vector sum of these distance vectors then resultant vector points to direction away from the obstacle as shown in fig 2. Thus, we have to move the robot in the direction of resultant motion vector pointing away from the obstacle.
{% include image.html url="/images/posts/obstacle.png" description="Fig 2. Obstacle Avoidance Behavior Implementation" %}  
IR range finder provides analog output voltage which decreases as distance to obstacle 
increases. Distance to obstacle (D) can be obtained from 
analog sensor voltage (V) as 

$$ D = \frac{139.33} {V − 1.0489} $$

Let distance to obstacle given by IR Range Finder placed 45 degrees apart in anticlockwise direction be represented as $$ d_i $$ for i = 1,2,...8. $$ D_{safe} $$ represents the safe distance between robot and obstacle so that we do not have to avoid that obstacle anymore. $$ R_{max} $$ represents the maximum range of IR Range Finder used.

---
**Algorithm: Goal-Seeking Behavior**
$$
\begin{align*}
& {\bf Inputs:}\ IR\ Sensor\ Readings\ (d_i), safe\_distance\ (D_{safe}), max\_range\ (R_{max})\\
& {\bf repeat}\\
& \ \ \ \ \ \ \ \ \ \ \
{\bf if}\ d_i > R_{max}\ {\bf then}\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ d_i\ = \ R_{max}\\
& \ \ \ \ \ \ \ \ \ \ \ {\bf end\ if}\\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ x\ and\ y\ component\ of\ obstacle\ distance\ as\ d_{ix} = d_i*\cos\left(30*i*\frac{\pi}{180}\right)\ and\ d_{iy} =  d_i*\sin\left(30*i*\frac{\pi}{180}\right)\\
& \ \ \ \ \ \ \ \ \ \ \
Obstacle\ avoidance\ motion\ vector\ U_{OAx} = \sum_{i=0}^7 d_{ix}\ and\ U_{OAy} = \sum_{i=0}^7 d_{iy} \\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ desired\ x,\ y\ and\ angular\ velocity\ as\ \dot{x} = K*U_{OAx},\ \dot{y} = K*U_{OAy}\ and\ \dot{θ} = 0.\\
& \ \ \ \ \ \ \ \ \ \ \
Obtain\ individual\ wheel\ velocities\ using\ inverse\ kinematics\ as\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 	
	\begin{bmatrix}
        V_1\\
        V_2\\
		V_3
	\end{bmatrix}
	=
	\begin{bmatrix}
        -\cos(\theta) & -\sin(\theta) & L\\
        \cos\left(\frac{\pi}{3}-\theta \right) & \sin\left(\frac{\pi}{3}-\theta \right) & L\\
		\cos\left(\frac{\pi}{3}+\theta \right) & \sin\left(\frac{\pi}{3}+\theta \right) & L
	\end{bmatrix}
	\begin{bmatrix}
        \dot{x}\\
        \dot{y}\\
		\dot{\theta}
	\end{bmatrix}\\
& \ \ \ \ \ \ \ \ \ \ \
Wheel\ velocities\ are\ maintained\ using\ encoder\ feedback\ and\ PID\ Controller,\ however\ some\ error\ exist.\\
& \ \ \ \ \ \ \ \ \ \ \
Update\ robot\ speed\ using\ actually\ measured\ individual\ wheel\ velocities\ (V_1',\ V_2',\ V_3')\ as\\
& \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 
	\begin{bmatrix}
        \dot{x'}\\
        \dot{y'}\\
		\dot{\theta'}
	\end{bmatrix}
	=
	\begin{bmatrix}
        -\frac{2}{3}\cos(\theta) & \frac{2}{3}\cos\left(\frac{\pi}{3}-\theta \right)& \frac{2}{3}\cos\left(\frac{\pi}{3}+\theta \right)\\
         \frac{-2}{3}\sin(\theta) & \frac{-2}{3}\sin\left(\frac{\pi}{3}-\theta \right) & \frac{2}{3}\sin\left(\frac{\pi}{3}+\theta \right)\\
		 \frac{1}{3L} & \frac{1}{3L} & \frac{1}{3L}
	\end{bmatrix}
	\begin{bmatrix}
        V_1'\\
        V_2'\\
		V_3'
	\end{bmatrix}\\
& \ \ \ \ \ \ \ \ \ \ \
Update\ robot\ position\ x=x+\dot{x'}*\delta t,\ y=y+\dot{y'}*\delta t\ and\ \theta = \theta + \dot{\theta'}*\delta t.\\
& \ \ \ \ \ \ \ \ \ \ \
\theta = \theta\ mod\ 2\pi\ as\ \theta\ \varepsilon\ [0,2\pi]\\
& {\bf until} d_i < D_{safe}\ for\ any\ i. 
\end{align*}\\
$$

---



---
title: My F1TENTH Journey — Lab 6, Pure Pursuit (Waypoint Tracker)
date: 2021-01-29 19:52:14
categories:
    - Projects
tags:
    - F1Tenth
    - F1/10 Racing
img: /images/f1tenth-lab6-cover.png
---

***This series of blogs marks the journey of my F1/10 Autonomous Racing Cars.***

***All my source codes can be accessed [here](https://github.com/shineyruan/F1Tenth_Labs).***

**Previous post:**
- [My F1TENTH Journey — Lab 1, Introduction to ROS](https://zhihaoruan.xyz/2021/01/24/f1tenth-lab1/)
- [My F1TENTH Journey — Lab 2, Automatic Emergency Braking](https://zhihaoruan.xyz/2021/01/25/f1tenth-lab2/)
- [My F1TENTH Journey — Lab 3, PID-Controlled Wall Follower](https://zhihaoruan.xyz/2021/01/27/f1tenth-lab3/)
- [My F1TENTH Journey — Lab 4, Reactive Planning Methods for Obstacle Avoidance](https://zhihaoruan.xyz/2021/01/27/f1tenth-lab4/)

<!-- more -->

## Overview 
In autonomous driving development, recording waypoints for future testing is very important. One way to record waypoints efficiently is only remembering the x, y, z coordinates of the vehicle regularly for some constant time gap. However, as ground vehicles are generally *non-holonomic* (i.e., the vehicle cannot make a turn in place), it is non-trivial to control an autonomous vehicle to exactly follow these pre-recorded waypoints. Hence, we need to design a path tracker (waypoint tracker) so that it can calculate and send commands to the vehicle to make it follow the pre-recorded waypoints. That is how "Pure Pursuit" algorithm comes into place.

## The Pure Pursuit Algorithm
The Pure Pursuit algorithm accounts for the non-holonomic constraints and the speed of the vehicle when it follows the waypoints. Given the current position of the vehicle, what it does is to always steer towards the next appropriate waypoint according to some lookahead distance $L$. In real practice, Pure Pursuit algorithm is often used with localization methods, i.e., a particle filter. 

But since the vehicle is non-holonomic, how can we steer towards the next waypoint properly? Setting the exact direction of the next waypoint from the current vehicle position is not feasible, as the vehicle is always in high speed and might overshoot the desired goal. One alternative option tries to interpolate the trajectory by finding an arc between the current position and the next waypoint (goal), which can be showed as the figure below.

![Arc interpolation between current position and goal.](/images/f1tenth-lab6-arc.jpg)

However there are infinitely many arcs between two points, and each of them has a different radius (curvature). Therefore we would like to add one more constraint to the arc: its center of radius must lie on the $y$ axis. In such way we can always find a unique arc interpolation for our trajectory, and it can be solved by simple math. After solving the geometry we can get the radius of the arc:
$$r=\frac{L^2}{2|y|}$$

![Finding the steering angle from the arc interpolation.](/images/f1tenth-lab6-steer.jpg)

Finally, we can simply set the steering angle proportional to the curvature of the arc: 
```cpp
double steering_angle = K_p * curvature;
```
`K_p` works exactly the same as a P controller and its value can also be tuned using PID tuning theory.

In summary, the Pure Pursuit algorithm can be implemented in the following steps:
1. Find the next waypoint (goal) from the current position and the list of all waypoints.
2. Interpolate an arc between current position and goal, and calculate its radius & curvature.
3. Set the steering angle proportional to the curvature.

## Actual Implementation


## Demo Videos
The Pure Pursuit algorithm with the vehicle running in 5m/s
<iframe width="560" height="315" src="https://www.youtube.com/embed/Qs2StzvzXHw" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


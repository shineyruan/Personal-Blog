---
title: My F1TENTH Journey — Lab 2, Automatic Emergency Braking
date: 2021-01-25 23:32:28
categories: 
    - Projects
tags:
    - F1Tenth
    - F1/10 Racing
---

***This series of blogs marks the journey of my F1/10 Autonomous Racing Cars.***

***All my source codes can be accessed [here](https://github.com/shineyruan/F1Tenth_Labs).***

**Previous post:**
- [My F1TENTH Journey — Lab 1, Introduction to ROS](https://zhihaoruan.xyz/2021/01/24/f1tenth-lab1/)

<!-- more -->

## Overview 
This lab focuses on implementing an AEB (Automatic Emergency Braking) in ROS node for F1/10 racing cars. AEB is widely used in autonomous vehicles as a basic safety guarantee to avoid collisions with objects.

The lab materials can be accessed [here](https://f1tenth-coursekit.readthedocs.io/en/stable/assignments/labs/lab2.html#). A PDF version is also attached below. 

{% pdf /pdf/f110_lab2.pdf %}

The lab was built on the *F1Tenth Simulator*, which can be accessed [here](https://f1tenth.readthedocs.io/en/stable/going_forward/simulator/sim_install.html).

## Demo
<iframe width="560" height="315" src="https://www.youtube.com/embed/vVHXqJv6NbY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Time-To-Collision
TTC (Time-To-Collision) is the time it would take for the vehicle to collide with an obstacle given its current heading and velocity. TTC can be calculated with the following format: 
$$TTC = \frac{r}{[-\dot{r}]_+}$$
where $r$ is the distance between vehicle and obstacle, $\dot{r}$ is its 1st derivative with respect to time, and the symbol $[x]_+$ denotes $\max(0,x)$.

In practice, we use LiDAR results to calculate TTC for each beam. Specifically, we project the current vehicle velocity onto the direction of each beam as $\dot{r}$, namely $\dot{r}=v\cos(\theta)$.

`sensor_msgs/LaserScan` provides us with LiDAR beams in all directions. In each `LaserScan` message, we have `angle_min`, `angle_max` and `angle_increment`, which corresponds to the *starting angle*, the *ending angle*, and the *step angle* between two adjacent laser beams. All the angles are given in the local coordinate frame (positive $x$ direction is the vehicle's heading) and positive $x$ direction is marked as angle value $0$.

We also have `float32[] ranges` in each `LaserScan` message, which gives us the distance to obstacle in the corresponding direction angle. Namely, `ranges[i] <--> angle_min + i * angle_increment`.

With all the information, we can write our AEB node as follows. 
```cpp
/**
 * @brief The class that handles emergency braking
 * 
 */
class Safety {
private:
    ros::NodeHandle n;
    double speed;
    const double deceleration;

    // ROS subscribers and publishers
    ros::Subscriber odom_subscriber;
    ros::Subscriber scan_subscriber;
    ros::Publisher publish_bool;
    ros::Publisher publish_ackermann;

public:
    Safety() : deceleration(8.26)  // F1/10 racing car deceleration: 8.26 m/s^2
    {
        n = ros::NodeHandle();
        speed = 0.0;

        // create ROS subscribers and publishers
        odom_subscriber = n.subscribe("/odom", 1, &Safety::odom_callback, this);
        scan_subscriber = n.subscribe("/scan", 1, &Safety::scan_callback, this);
        publish_bool = n.advertise<std_msgs::Bool>("/brake_bool", 1);
        publish_ackermann = n.advertise<ackermann_msgs::AckermannDriveStamped>("/brake", 1);
    }

    void odom_callback(const nav_msgs::Odometry::ConstPtr &odom_msg) {
        // update current speed
        geometry_msgs::Vector3 velocity = odom_msg->twist.twist.linear;
        speed = lab2_utils::vecLength(velocity);
    }

    void scan_callback(const sensor_msgs::LaserScan::ConstPtr &scan_msg) {
        unsigned int N = scan_msg->ranges.size();
        for (unsigned int i = 0; i < N; ++i) {
            double range = scan_msg->ranges[i];
            if (range > scan_msg->range_max || range < scan_msg->range_min) continue;

            double angle_to_heading = scan_msg->angle_min + i * scan_msg->angle_increment;
            double projected_speed = std::max(-1 * speed * std::cos(angle_to_heading), 0.0);

            // calculate TTC
            double ttc = (projected_speed <= 0.0)
                             ? std::numeric_limits<double>::infinity()
                             : range / (projected_speed);

            // publish drive/brake message
            if (ttc <= 2.5 * speed / deceleration) {  // 2.5 is a parameter (magic number) from simulation experiments
                std_msgs::Bool msg_bool;
                msg_bool.data = true;

                ackermann_msgs::AckermannDriveStamped msg_brake;
                msg_brake.drive.speed = 0.f;

                publish_bool.publish(msg_bool);
                publish_ackermann.publish(msg_brake);

                ROS_INFO("Emergency brake message sent");
            }
        }
    }
};
```

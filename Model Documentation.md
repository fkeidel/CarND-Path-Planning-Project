# Overview
Each time the path planner is running, the path planner is returning 50 trajectory points to the simulator (line 452-453). Since the simulator is consuming one point every 0.02 seconds, this means that the path is reaching 1 second (50*0.02 sec) into the future.
Each time the simulator runs, it consumes only some points of the trajectory and returns the list of unconsumed points (line 275-276). The path planner reuses the unconsumed points (line 409-415) and adds missing points so that it is returning 50 points again (line 425-444).
# Detailed explanation
In line 297, the number of unconsumed points is evaluated. If the simulator has returned unconsumed points, the s value is set to the s value of the last unconsumed point (line 306-309).

In lines 311-345 breaking and lane changing is handled.

First, the region in front of the car is checked. If there is a car in a protected region of 30 m in front of the car (line 313), a flag is set, that the car is too close to another car (line 317). Furthermore, the path planner checks, if the left lane or the right lane has enough empty space to change the lane. If a protected area of plus/minus 30 meter is free on the left or right lane, a lane change is requested (lines 319-329). If there is not enough empty space in the left or right lane, the car stays in it's lane (line 331).

The lines 334-345 handle increasing and decreasing speed. The car increases speed step by step until it reaches a desired maximum velocity, which was selected to be a little bit lower as the speed limit. The car decreases speed step by step, if a slower car in front is too close.

In the lines 347-444 the trajectory is created. Since it is easier to create a trajectory in a local car coordinate system, at first, the reference location and orientation of the local coordinate system is set to the current car's location and orientation. 

When the car starts driving, the first 2 points of the trajectory are chosen to lie in line with the orientation of the car (356-365)

In the following iterations, to get a smooth transition between the iterations, the first 2 points of the trajectory are chosen to be the last 2 points from the previous path from the last iteration (368-380).

Then the trajectory gets created by sampling points from a spline. The knots of the spline are the first 2 points created above to get a smooth transition and 3 additional points lying on the target lane. The target lane is the same lane, if keeping lane, or another lane, if changing lane is requested (382-392).

The knots get transformed to the local car coordinate system (395-403). The spline is created from the knots (406-407). A sampling step dx is calculated in a way that the car will drive with the desired speed (417-421) and points from the spline are sampled (425-428) until the number of desired points (=50) is reached.  The sampled points get transformed back to the global coordinate system (432-440) and get inserted into the list of the trajectory points (next_vals) that are sent to the simulator (442-443).

# Result
The path planner was able to drive the car without incident for more than a 4.32 miles. The following image is showing the car after 10 miles.

![10 miles driven](https://github.com/fkeidel/CarND-Path-Planning-Project/blob/master/img/10%20miles%20driven.PNG "10 miles driven.PNG")

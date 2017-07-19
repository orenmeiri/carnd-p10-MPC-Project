# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

## Solution Implementation

###The Model

The model predicts 2 actuators: acceleration/breaking AND steering angle for driving a car around the circuit.

At any given time, the simulator provides the following states:
x, y = current vehicle position of the car in simulator map coordinates
speed = current vehicle  speed
psi = current vehicle  angle relative to the map
ptsx, ptsy = road waypoints ahead of the car.


steps:

1. Translates the waypoints to car coordinates and current orientation (psi).
2. Calculate coefficients of 3rd degree polynomial of track ahed
3. Calculate current CTE using coefficients to calculate y of x = 0
4. Calculate current EPSI by measuring angle with equation coefficients. Current heading is 0.
5. Calculate position at t + 0.1 sec.  X is measured using current speed. For simplicity assume vehicle will continue in straignt line and thus y will be 0.
6. Feed current state to MPC.
7. MPC configures these parameters for passing to Ipopt.
	1. Vars lowerbound & upperbound - limits vars to possible ranges: x, y, v, psi, cte, epsi - any value. delta (steering angle) -/+ 25 degrees. a (acceleration) -/+ 1
	2. Constraints lowerbound & upperbound - limit so that decided params that dictate specific route for the car much match that route.
8. Call ipopt to calculate best solution.  
9. operator() is used to:
	1. Calculate cost function from cte, epsi, diff of speed from desired speed, exessive use of accurators, exessive range to actuator values.
	2. Calculate predicted states (x, y, v, psi, cte, epsi from actuators to ensure constraints set above are are not broken.
10. e
11. Extract from result current actuators (steering angle & acceleration) and future predicted positions from current path.
11. Send results to simulator

###Timestep Length and Elapsed Duration (N & dt)

My solution includes these set values:
N = 10
dt = 0.3

These values were set by trial and error.  Larger N value of 20 resulted in long computation time.

For dt, I tried 0.1 but together with N = 10, calculated 1 second into the future.  I found this too short timescale which resulted in unstable control.  The value 0.3 (* 10) is 3 seconds which enabled me to calculate path 3 seconds into the future and better results.

### Polynomial Fitting and MPC Preprocessing

The simulator provides waypoints (and car state) in map coordinates.  To simplify calcultions and avoid problems with points around y axis these were transfered to car coordinate system.

Each waypoint was translated (moved) relative to car position and rotated relative to car psi (orientation).

Similarly the car position sent to the model is relative to the same coordinate system origin.  Apart from predicting position in 0.1 sec due to delay, the car position is 0,0 and psi = 0.


### Model Predictive Control with Latency

The model predicts vehicle state 0.1 seconds in the future.  A simple approach is taken for calculating position
x' = x + v * 0.1
y' = y (assume car is not rotating and this will not move in y axis)



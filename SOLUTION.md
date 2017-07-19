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
	
	   x[t+1] = x[t] + v[t] * cos(psi[t]) * dt
	   
      y[t+1] = y[t] + v[t] * sin(psi[t]) * dt
      
      psi[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
      
      v[t+1] = v[t] + a[t] * dt
      
      cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
      
      epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt

10. Extract from result current actuators (steering angle & acceleration) and future predicted positions from current path.
11. Send results to simulator

###Timestep Length and Elapsed Duration (N & dt)

My solution includes these set values:
N = 10
dt = 0.15

These values were set by trial and error.  Larger N value of 20 resulted in long computation time.

I tried 0.1, 0.15 and 0.3 dt.  I found that larger values predict smoother and more stable route.  It was suggested to me to avoid 0.3 as it calculates too much into the future so I reduced to 0.15 sec.

### Polynomial Fitting and MPC Preprocessing

The simulator provides waypoints (and car state) in map coordinates.  To simplify calcultions and avoid problems with points around y axis these were transfered to car coordinate system.

Each waypoint was translated (moved) relative to car position and rotated relative to car psi (orientation).

Similarly the car position sent to the model is relative to the same coordinate system origin.  Apart from predicting position in 0.1 sec due to delay, the car position is 0,0 and psi = 0.


### Model Predictive Control with Latency

The model predicts vehicle state 0.1 seconds in the future.  All 6 state variables are computed by equations similar to those in 9.2



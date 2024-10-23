
##Overview of the Process##

Suppose you have a Process (e.g. a temperature chamber with heater and compressor) which produces a measurable Process Variable y (e.g. the temperature measurement in the chamber).  

The Process is controlled via a drive signal u that comes from the controller, and your goal is to match the PV to a target value, also known as the Setpoint, or y$_{sp}$

PID stands for Proportional, Integral, and Derivative.
- Proportional: take the error and multiply it by a constant K$_p$
- Integral: take the error and multiply it by a constant K$_i$
- Derivative: take the rate of change of the error and multiply it by a constant K$_d$
	- how the error changed between this measurement and the last measurement

The official math version of it looks like this. \
![Image Description](Pasted%20image%2020240701123450.png)

![Image Description](Pasted%20image%2020240701125949.png)

Each cycle, the PID controller calculates the next drive signal based on these three terms. 
The performance of a PID controller depends heavily on whether the appropriate PID parameters have been selected. 

PID constants can be tuned using a variety of methods. See bottom of this document.

There's another way we can formulate the PID formula.

![Image Description](Pasted%20image%2020240701133326.png)


There are several reasons for using this alternate formulation:

1. This allows for the constants _Ti_ and _Td_ to be expressed in units of time (in fact, these constants are sometimes known as the “Integral time” and “Derivative time”, respectively)
2. _Kp_ can be interpreted as the overall “gain” of the PID controller, with increases or decreases to _Kp_ being fairly applied to the integral and derivative terms as well
3. The Ziegler-Nichols and Åström-Hägglund tuning methods both use this form for their parameter recommendations

**Note:** the _Ti_ (integral time) constant is in the denominator of the formula. This means that increasing _Ti_ will decrease the integral contribution to the PID controller output, while decreasing _Ti_ will increase the integral contribution



However, in actual control, we end up using the discrete form.

![Image Description](Pasted%20image%2020240701133351.png)

e$_k$ is the ol $y_{sp}[k]$ - $y[k]$  
T is the sampling interval


Note that the integral and derivative terms are multiplied and divided by the corresponding sampling interval respectively.

Lets get into each of these components to help us understand the system.

### Proportional Controllers

Suppose you began with the simplest type of controller, a proportional controller.

$u[k] = K_p * e[k] = Kp (y_{sp} [k] - y[k])$

This control adjusts the existing output in proportion to the current error measured.

Below are some simulated tests with various _Kp_ values (0.1, 0.5, 0.7) for an open air temperature controller.

![Image Description](Pasted%20image%2020240701135511.png)

Takeaways
 1. A higher _Kp_ will help the Process Variable reach the Setpoint at a quicker pace, as shown comparing the first two plots
2. Too high of a _Kp_ will result in uncontrollable oscillations, as shown in the last plot
3. Even with a balanced _Kp_ constant, there is always some static error, which will depend on the process being controlled.



## Integral Term

The purpose of introducing the Integral component is to address the static error in the process.

Imagine a helicopter aiming to reach a certain elevation and youre controlling rotor 
speed. Suppose you want it to be 100m. You get to 100m, the rotor turns off, so now ur falling, then you turn the rotor back on, so you rise up again, and you'd endlessly oscillate. 

Basically, bad things happen to proportional control when e = 0, so we guarantee some level of output in the e = 0 scenario with the integral term. This means that over time, eventually the helicopter will stably stay at the exact height it should.

Insights and takeaways:

1. Increasing either the Proportional or Integral component will help the Process Variable converge more quickly to the Setpoint, with the risk of oscillations if increased too much  
    Reminder: increasing the Integral component is done by lowering the Ti constant
2. Increasing the Proportional component means the error sum grows less quickly, because there is less time to accumulate error

## Derivative Control

The final component of PID control is the Derivative component, which is a multiplier based on the rate of change in the error.

The purpose of Derivative control is to provide a “dampening” effect to limit overshoot and converge more quickly upon the Setpoint. It predicts future error and compensates the output ahead of time.

Adding Derivative control will reduce the amount of overshoot. However, after a certain point, increasing the Derivative no longer improves the overshoot and instead creates oscillations.

## Integral Windup

One of the most common pitfalls with PID control is dealing with Integral windup. 

Integral windup is when you accumulate too large of an error sum when approaching the Setpoint from far away. It results in a significant amount of overshoot and inefficient control.  
  
Note that the Integral component is the only part of the PID control process with long-term memory. While the Proportional and Derivative components help react to live errors, the Integral component is the only factor for improving the control quality over time.

 Given that the Setpoint is pre-determined, the controller can only affect the error sum by moving the Process Variable itself.

If the controller moves the Process Variable past the Setpoint onto the other side, this is called “overshoot”.

Overshoot is necessary. As long as the process variable is on one side of the setpoint, the error sum is monotonically increasing or decreasing. In almost all cases, the error sum will not be the ideal one (the ideal one being the one that lets the helicopter rotate perfectly in place) and will cross over the barrier, which means it must go back down in order to correct the error sum.

PID variables are here to reduce the severity of the overshoot rather than remove it entirely. 

By the way, in general, the further you start from the set point initial condition wise, the worse the over shoot is. This makes sense, because the farther we are from the set point, the longer timeframe there is for the "integral windup" to build up. 

How can we combat integral windup?
Imagine a simple rule, where the Integral error accumulation is disabled when the output is saturated (i.e. the output exceeds its maximum or minimum limits). That'll get your head cooking.

Common anti-windup measures include:

1. Disabling the error sum accumulation when the controller output is saturated
2. Disabling the error sum accumulation until the Process Variable is within a certain range (known as the “controllable region) of the Setpoint
3. Resetting the accumulated error sum to zero (or another preset value from previous test runs) when the Process Variable first crosses the Setpoint
4. Clamping - Limiting the integral term so it doesn't exceed a pre-defined threshold. This prevents the integral term from growing too large and causing windup.
5. Back Calculation - Adjusting the integral term based on the difference between the saturated and unsaturated control signals, effectively compensating for the saturation.

## PID tuning
The most important part of configuring the PID controller is selecting the PID constants _Kp_, _Ti_, _Td_.  This process is known as “tuning” the PID controller.



[Resource on PID Tuning Methods](https://eng.libretexts.org/Bookshelves/Industrial_and_Systems_Engineering/Chemical_Process_Dynamics_and_Controls_(Woolf)/09%3A_Proportional-Integral-Derivative_(PID)_Control/9.03%3A_PID_Tuning_via_Classical_Methods)
PID Tuning Methods:
- Ziegler-Nichols
- Cohen-Coon
- Åström-Hägglund

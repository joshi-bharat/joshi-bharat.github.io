---
title: DC Motor Speed Control using Encoder Feedback
date: 2016-09-12
collection: posts
permalink: /posts/encoder-feedback
tags:
    - motor-control
    - PID
    - mobile-robot
---

The speed of a DC motor is proportional to the voltage applied to the motor. When using digital control, a pulse-width modulated (PWM) signal is used to generate an average voltage. The motor winding acts as a low pass filter, so a PWM waveform of sufficient frequency will generate a stable current in the motor winding. Though speed is proportional to PWM duty cycle, in systems where precise speed control is required, some sort of feedback mechanism must be included. Optical incremental encoders are widely used to provide position measurements in control systems at fixed sampling frequency. Encoder feedback implemented in combination with PID controller can be used to regulate the DC motor speed in response to load disturbances and noise. PID controller has the ability to eliminate steady-state offsets through integral action, correct present error through proportional action, and it can anticipate the future through derivative action. The objective of a PID controller in a DC motor speed control system is to maintain speed at a given reference value and be able to accept new set point values dynamically.

### Incremental Optical Encoder ###
Optical Encoders are widely used to measure the speed of DC Motor.Incremental encoders consist of three basic components: a slotted disc, a light source and a dual light detector, as shown in Figure 1. The light source shines on the disc, which has a regularly spaced radial pattern of transmissive and reflective elements, called encoder increments. The quadrature light detector measures the amount of light
passed through the slotted disc and generates two quadrature output pulse signals, denoted by A and B. The up and down changes of the pulse signals are counted as a measurement of the encoder position.
{% include image.html url="/images/posts/encoder_fig.png" description="Fig 1. Mechanism of incremental optical quadrature phase encoder " %}
For the application of feedback control to motion systems with optical incremental encoders, the position is generally measured at a fixed sampling frequency. To improve velocity information obtained from optical incremental encoders; fixed time methods are used, which measures the traveled encoder counts over a fixed period-time. Counts are taken at particular sampling interval and try to control speed of DC motor to some reference input at that sampling time.

An encoder with one set of pulses is sometimes not sufficient because it cannot indicate the direction of rotation. Using two code tracks with sectors positioned 90 degree out of phase (as shown in Figure 2); the two output channels of the quadrature encoder indicate both position and direction of rotation. For example, if A leads B, the disk is rotating in a clockwise direction. If B leads A, the disk is rotating in a counter-clockwise direction. Therefore, by monitoring both number of pulses and the relative phase of signals A and B, the microcontroller can track both the position and direction of rotation. In addition, some quadrature encoders include a third output channel – called a zero or reference signal – which supplies a single pulse per revolution. This single pulse can be used for precise determination of a reference position. This signal is called the Z-Terminal or the index in most of encoder.
{% include image.html url="/images/posts/quadrature-signal.gif" description="Fig 2. Typical Encoder Quadrature Signal" %}
Encoder pulses during a sampling interval can be converted to motor speed using the equation:

$$
\begin{align*}
  & C_m = \Pi D_n/nC_e \\
  & where, \\
  & C_m =  conversion\ factor\ that\ translates\ encoder\ pulses\ into\ linear\ wheel\ displacement \\
  & D_n = nominal\ wheel\ diameter\ (in\ m) \\
  & C_e = encoder\ resolution\ (in\ pulses\ per\ revolution),\\
  & n = gear\ ratio\ of\ the\ reduction\ gear\ between\ the\ motor\ (where\ the\ encoder\ is\ attached)\ and\ the\ drive\ wheel
\end{align*}
$$

### Motor Control Signal ###
Instead of generating an analog output signal with a voltage proportional to the desired motor speed, it is sufficient to generate digital pulses at the full system voltage level (for example 5V). These pulses are generated at a fixed frequency, for example 20 kHz, so they are beyond the human hearing range. By varying the pulse width,  we also change the equivalent or effective analog motor signal and therefore control the motor speed. One could say that the motor system behaves like an integrator of the digital input impulses over a certain time span.
{% include image.html url="/images/posts/pwm.png" description="Fig 3. PWM Signal" %}

$$
\begin{align*}
  Duty\ Cycle(\delta) = \frac{Pulse\ On Time(\delta T)}{Pulse\ Time\ Period(T_{PWM})}
\end{align*}
$$

The relation between average voltage, supply voltage and duty cycle is given by:

$$
\begin{align*}
  V_{average} = \delta * V_{supply}
\end{align*}
$$

The average voltage applied to the motor is proportional to the PWM duty cycle. This property of PWM can be used to smoothly vary the speed of the motor by changing the average value which is equivalent to changing the duty cycle of the signaling pulse.

### PID Controller ###
The PID tuning comprises of three parameters: P (proportional), I (integral) and D (differential) terms whose proper values maintain the stability of the whole system. A proportional controller (with proportional gain KP) will have the effect of reducing the rise time and will reduce, but never eliminate, the steady-state error. An integral control (with integral gain Ki) will have the effect of eliminating the steady-state error, but it may make the transient response worse. A derivative control (with differential gain Kd) will have the effect of increasing the stability of the system, reducing the overshoot, and improving the transient response.
{% include image.html url="/images/posts/pid.png" description="Fig 4. Block Diagram of PID Controller" %}
Combining proportional, integral and derivative control, PID controller can be expressed mathematically as:

$$
\begin{align*}
  u(t) = K_pe(t)+K_i\int_0^te(t)d(t)+K_d\frac{de(t)}{dt}
\end{align*}
$$

Digital Model of PID can be expressed as:

$$
\begin{align*}
& u(n) = K_pe(n)+K_iT_s\sum_0^Ne(n)+K_d\frac{e(n)-e(n-1)}{T_s}
\end{align*}
$$

### Speed Control Implementation ###
The experimental setup consists of mechanical setup for brushed DC Motor along with feedback and motor driver circuitry (H-bridge) as shown in Figure 5.
{% include image.html url="/images/posts/control_system.png" description="Fig 5. Experimental Setup for DC Motor Speed Control" %}
Discrete PID controller and encoder interfacing are implemented on ATMega8 microcontroller which
performs encoder data acquisition at a given sampling frequency. A discrete PID controller reads the error, calculate and output the control input at a given time interval, at the sample period. The sample time was chosen less than the shortest time constant in the system. Here, the reference signal r[n] indicates the desired motor speed where positive value indicates anticlockwise rotation and negative value indicates clockwise rotation. The quadrature input signals from optical encoder are fed to interrupt driven AtMega8 external interrupt pins to capture encoder events. The microcontroller counts no of encoder pulses in the sampling interval and calculates motor rotational speed in Hz as

$$
\begin{align*}
& Speed(in Hz) = \frac {Count}{PPR * T} \\
& where, \\
& Count = No.\ of\ Counts\ in\ Sampling\ Interval \\
& PPR = Encoder\ Resolution\ (Pulses\ per\ Revolution) \\
& T = Sampling\ Interval
\end{align*}
$$

Error at each sample instant is calculated and applied to PID controller which calculates the desired control variable in the form of PWM. Thus obtained PWM signal is applied to motor driver (H-bridge) that actuates the DC motor in such a way that motor speed is proportional to the PWM duty cycle. However, PID parameters (Kp, Ki and Kd) must to tuned for desirable system response and were tuned manually by observing the effect of change in PID constants on the system performance.

**Integral Windup**

Motor has some speed limit and control variable (PWM) may reach the DC motor speed limit. When this  happens  the  feedback loop  is  broken  and  the  system  runs  as an open  loop  because  the  motor  remains  at  its  limit independently  of  the process  output as long as the actuator is saturated. In PID controller the error may be continue to be integrated and  the  integral  term  may  become  very  large  or, colloquially, it  "winds  up". The consequence  is  that  any  controller  with  integral  action may  give large  transients  when  the  motor  saturates. Also high integral term results in sluggish PID controller response.
{% include image.html url="/images/posts/integral_windup.png" description="Fig 6. Illustration of Integral Windup" %}
Figure 6. shows the control signal, the measurement signal, and the set point in a case where the control signal becomes saturated. After the first set-point change, the control signal increases to its upper limit umax. This control signal is not large enough to eliminate the control error. Therefore, the integral of the control error, and the integral part of the control signal, increases. Since the desired control signal u increases, there is a difference between the desired control signal and the true control signal uout. Figure 6. also shows what happens when after a certain time the set point is lowered to a level where the controller is able to eliminate the control error. Since the sign of the control error becomes negative, the control signal starts to decrease, but since the desired control signal u is above the limit umax, the true control signal uout is stuck at the limit for a while and the response becomes
delayed.
Simple integral anti-windup strategy can be implemented by limiting the integral term between
*I_Term_min* and *I_Term_max*.

**Derivative Filtering**

A caveat to derivative action is that an ideal derivative has very high output for high-frequency signals. This means that high-frequency measurement noise will generate large variations of the control signal. The effect of measurement noise to some extent can be eliminated by first order filtering of derivative term.  
The complete digital PID algorithm can be implemented as:

---
**Algorithm: PID Control**
$$
\begin{align*}
& {\bf Inputs:}\ desired\_velocity(des\_vel[n])_,\ measured\_velocity(mes\_vel[n])\\
& {\bf Output:}\ Control(PWM)\ Signal\ to\ H-Bridge(u[n])\\
& error[n]\ =\ des\_vel[n]\ -\ mes\_vel[n]\\
& P\_Term[n]\ = K_p*error[n]\\
& I\_Term[n]\ = I\_Term[n-1]+\ K_i*error[n]\\
& D\_Term[n]\ = (1-\alpha)*D\_Term[n-1]+\alpha*K_d*(y[n]-y[n-1])\\
& I\_Term[n]\ = 
\begin{cases}
I\_Term_{min},  & if\ I\_Term[n]<I\_Term_{min}\\
I\_Term_{max},  & if\ I\_Term[n]>I\_Term_{max}
\end{cases}\\
& u[n]\ =\ base\_PWM + P\_Term[n]+D\_Term[n]+I\_Term[n]  
\end{align*}
$$

---

The closed loop response of the speed control system using step input for various PID parameters is illustrated in fig. 7, 8 and 9.
{% include image.html url="/images/posts/overdamped.jpg" description="Fig 7. Overdamped Response" %}
{% include image.html url="/images/posts/overshoot.jpg" description="Fig 8. Response with Overshoot" %}
{% include image.html url="/images/posts/optimum.jpg" description="Fig 9. Near Optimum Response" %}

#### References ####

1. Bharat Joshi, Rakesh Shrestha, Ramesh Chaudhary. “Modeling, Simulation and Implementation of Brushed DC Motor Speed Control Using Optical Incremental Encoder Feedback”, Proceedings of IOE Graduate Conference, 2014 - [[pdf]](http://conference.ioe.edu.np/ioegc2014/papers/IOE-CONF-2014-66.pdf)


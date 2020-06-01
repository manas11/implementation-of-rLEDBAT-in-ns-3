# Implementation-of-rLEDBAT-in-ns-3
-----------------------
## Course Code: CO365 
## Course Name: Advanced Computer Networks
-------------------------
### Overview

rLEDBAT, a receiver-based, less-than-best-effort congestion control algorithm. rLEDBAT is inspired in.<br/>
LEDBAT but with the following differences:
* rLEDBAT is implemented in the TCP receiver and controls the sending rate of the sender through the TCP Receiver Window.
* rLEDBAT uses the round-trip-time (RTT) to estimate the queuing delay.
* rLEDBAT uses an Additive Increase/Multiplicative Decrease algorithm to achieve inter-(r)LEDBAT fairness and avoid the     latecomer advantage observed in LEDBAT.
 
 * rLEDBAT limits the queueing delay in the path to a target delay T.
 * rLEDBAT uses the RTT to estimate the queueing delay.  
 * The rLEDBAT receiver uses the TCP TimeStamp option to measure the RTT.
 * rLEDBAT estimates the Base RTT (i.e. the RTT when there is no queuing delay) as the minimum observed RTT in the last n minutes. 
* rLEDBAT estimation of the queuing delay (qd) is obtained subtracting the Base RTT from latest sample(s) of the RTT.

Suppose that the rl.WND was last updated at time t0 and its current value is then rl.WND(t0) and at time t1 a packet is received.The rLEDBAT receiver updates rl.WND as follows:
```
if qd < T, then rl.WND(t1) = rl.WND(t0) + alpha*MSS/rl.WND(t0)

if qd > T, then rl.WND(t1) = rl.WND(t0)*betad

with MSS being the Maximum Segment Size of the TCP connection, and alpha and betad being the additive increase and multiplicative decrease parameters respectively
```

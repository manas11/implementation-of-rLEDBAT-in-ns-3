# Implementation-of-rLEDBAT-in-ns-3

## Course Code: CO365 
## Course Name: Advanced Computer Networks

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

alpha > 1 
0 < betad < 1

with MSS being the Maximum Segment Size of the TCP connection, and alpha and betad being the additive increase and multiplicative decrease parameters respectively

```
* The multiplicative reduction is applied at most one per RTT.

### Controlling Receiver Window

* rLEDBAT uses the Receive Window (RCV.WND) of TCP to enable the receiver to control the sender's rate.

* RCV.WND is used to announce the available receive buffer to the sender for flow control purposes.

* fc.WND - flow Controll Reciever Window as proposed by RFCRFC793bis (in simple terms its the standard TCP reciever window followed in New Reno or most of the TCP's).

* rl.WND = Reciver window as proposed by rLEDBAT draft(this needs to be calculated using the LEDBAT equation replacing the one-way delay by RTT delay).
```
Thus
RCV.WND = min(fl.WND, rl.WND)
```
```
SND.WND = min(cwnd , RCV.WND)
```

### Avoiding Window Shrinking
 * The rLEDBAT algorithm increases or decreases the rl.WND according to congestion signals
 (variations on the estimations of the queueing delay and packet loss).
 
 * If the new congestion window is smaller than the current one, we progressively reduce the advertised receiver window to avoid packet drops.
 ```
 For example, if the current congestion window is 500, the next congestion window is 300. Instead of directly advertising 300 as the receiver window, we will subtract the bytes acknowledged(received in this case), hence avoiding the packet drop.
 ```
 
 * The rLEDBAT algorithm only allows to perform at most one multiplicative
   decrease per RTT, this allows the receiver to drain enough packets
   from the packets in-flight to reach the reduced window resulting form
   the rLEDBAT algorithm without need for resorting to shrinking the
   receiver window.
   
### Window Scale option
   
 * The rLEDBAT client should set WS option values lower than 12.(This is as per the recommendation made in the rLEDBAT draft)
 
### Using the RTT to estimate the queueing delay

```
This section needs to be updated because as per the draft there is no algorithm available.
We can try using the queueing delay calculation algorithm by replacing the one-way delay by RTT. (Not sure how to do so placing it in Things need to be done)
```

### Inter-rLEDBAT fairness

* In rLEDBAT the congestion window is decreased by a multiplicative factor betad when the measured queueing delay is larger than the target T.

* AIMD reduces the large flows rapidly hence allowing space for new flows.

### Reacting to packet loss

### ALgorithm
The  Below algorithm assumes that we don't use WS option because if we use that and it get more than 11 then the nature of rLEDBAT is not known.
Also according to the draft a value less than 11 is suggested.

```
on initialization
       DRAINED.BYTES = 0
       base_RTTs set to maximum value
       current_RTTs set to maximum value
       rl.WND set to max value
       end.reduction.time = 0
```
```
on packet arrival
    DRAINED.BYTES = DRAINED.BYTES + SEG.LEN
    RTT calculation
        SEG.RTT = SEG.Time - SEG.TSE (the new sample of the RTT is the time of
        arrival of the segment minus the time at which the segment containing
        the TSVal value was issued)
        Update current_RTTs with SEG.RTT (substitute the oldest RTT sample in
        the current_RTTs array by SEG.RTT)
        Update base_RTTs with SEG.RTT (store SEG.RTT in the current current
        minute position, if SEG.RTT is smaller than the value in that
        position)
```
```
QD = min(current_RTTs) - min(base_RTTs)
```
```
If local.time > end.reduction.time then
        If SEG.SEQ < RCV.HGH AND SEG.TSE > TSE.HGH then
            rl.WND = max(rl.WND*betal, 1)
            end.reduction.time = local.time + min(current_RTTs)
        else
            If QD < T, then rl.WND = rl.WND+ alpha*MSS/rl.WND
            else QD > T, then rl.WND(t1) = max(rl.WND*beta1, 1)


          on sending a packet
      if rl.WND > rl.WND.WS or (rl.WND.WS - rl.WND) < DRAINED.BYTES then
          rl.WND.WS = rl.WND
      else
          rl.WND.WS = rl.WND.WS - DRAINED.BYTES
      DRAINED.BYTES = 0
      RCV.WND = min(fc.WND, rl.WND.WS)
```
### Parameters 
```
T: Target delay

      betal: multiplicative decrease factor in case of packet loss

      betad: multiplicative decrease factor in case of RTT exceeds T

      alpha: additive increase factor.
```

### Variables
```
current_RTTs is an array with the last k measured RTTs

      base_RTTs is an array with the minimum observed RTTs in the last n
      minutes

      RCV.SEQ is the sequence number of the last byte that was received
      and acknowledged

      RCV.HGH is the highest sequence number of a received byte (which
      may not have been acknowledged yet)

      TSE.HGH is the TSecr value contained in the segment containing the
      byte with sequence number RCV.HGH
      
      SEG.SEQ is the sequence number of the incoming segment

      SEG.TSE is the TSecr value of the incoming segment

      SEG.time is the local time at which the incoming segment was
      received

      SEG.RTT is the latest sample of the RTT

      QD latest estimation of the queueing delay

      rl.WND window calculated by rLEDBAT without taking into account
      the window shrinking avoidance constraints

      rl.WND.WS window calculated by rLEDBAT after taking into account
      the window shrinking avoidance constrains

      DRAINED.BYTES number of bytes drained from the flight-size since
      the last packet sent

      fc.WND window calculated by standard TCP receiver

      end.reduction.time auxiliary variable used to prevent rl.WND from
      being updated after a window reduction
```

 NOTE:- The proposed algorithm has been derived after going through the draft algorithm may change and some other variable might be used when actually implemting in NS-3.
 
 ### TODO Task
 * How to solve synchronisation problem
 * Modify the existing LEDBAT NS-3 code to make it rLEDBAT instead of writing it from scratch

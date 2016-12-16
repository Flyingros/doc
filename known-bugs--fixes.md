# Bugs & fixes

## Mavros 0.18.4 - ENU &lt;-&gt; NED bug.

Sending ENU informations to `/mavros/vision_position/pose` do lead to a yaw issue. Indeed, the **ENU to NED** conversion does a wrong yaw rotation. 

The _solution _if we can call it so, is to send a** +90°** rotated yaw position _and _a **+90° **rotated yaw setpoint. 

The second solution is to use raw mavlink messages.




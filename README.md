# SO-101 Robotic Arm

![Build](https://img.shields.io/badge/build-passing-brightgreen)

Lerobot So-101 Robotic Arm

Object is to connects two robot arms:

Leader → the arm you physically move
Follower → the arm that mimics the leader’s movements

The Python run_teleop.py script:

Creates configuration objects for each robot

Connects to both robots over serial ports

Enters a loop running at 50 Hz

Reads the leader’s joint positions

Sends those positions directly to the follower

Stops cleanly when you press Ctrl+C

Let's break down the script for clear understanding and chocies made:
 ```python
import time
import signal

time → used for timing the control loop
signal → used to catch Ctrl+C (SIGINT) so the program can shut down gracefully

```python
from lerobot.teleoperators.so_leader.config_so_leader import SOLeaderTeleopConfig
from lerobot.teleoperators.so_leader.so_leader import SO101Leader

Imports the leader robot configuration class and the leader robot interface.

```python
from lerobot.robots.so_follower.config_so_follower import SOFollowerRobotConfig
from lerobot.robots.so_follower.so_follower import SO101Follower

Same idea, but foe the follower robot.

# Control Loop Setting
```python
CONTROL_RATE = 50.0
DT = 1.0 / CONTROL_RATE

The loop should run at 50 Hz
DT is the time per loop interation-> 0.02 seconds

# Running Flag
```python
running = True

A global boolean used to stop the loop when Ctrl+C is pressed.

# Interrupt Handler
```python
def handle_interrupt(sig, frame):
    global running
    print("\n[teleop] Stopping…")
    running = False

(This function is called when the user presses Ctrl+C)
It sets = False, which will break the main loop.
```python
signal.signal(signal.SIGINT, handle_interrupt)
Registers the handler so the program catches Ctrl+C instead of crashing

# main() Function
# Create Leader Config
```python
print("[teleop] Creating leader config…")
leader_cfg = SOLeaderTeleopConfig(
    port="/dev/tty.usbmodem5AE60555481",
    use_degrees=False,
)

Creates a configuration object for the leader robot
Specifies which USB serial Port it's connected to 
use_degrees=False -> joint angles will be in radians

# Create Follower Config
```python
print("[teleop] Creating follower config…")
follower_cfg = SOFollowerRobotConfig(
    port="/dev/tty.usbmodem5AE60558601",
    use_degrees=False,
    cameras={},
)
Same idea, but for the follower robot.
cameras={} means no cameras at this time

# Initialize Robot Objects
```python
print("[teleop] Initializing leader and follower…")
leader = SO101Leader(leader_cfg)
follower = SO101Follower(follower_cfg)

creates actual robot interface objects using the configs.

# Connect to Robot
```python
print("[teleop] Connecting leader…")
leader.connect()
```python
print("[teleop] Connecting follower…")
follower.connect()

Opens communication with each robot.

# User Instructions
```python
print("[teleop] Teleoperation running.")
print("         Move the leader arm to control the follower.")
print("         Press Ctrl+C to stop.\n")

Just console output

# Main Control Loop
```python
while running:

Runs until Ctrl+C sets running = False

# Start Loop Timer
```python
loop_start = time.time()

Used to maintain the 50 Hz rate.

Read Leader Action
```python
action = leader.get_action()

This is the key step:
Reads the leader's current joint positions
Returns a dictionary like:
```python
{
    "shoulder_pan.pos": 0.12,
    "shoulder_lift.pos": -0.45,
    ...
}

# Send Action to Follower
```python
follower.send_action(action)

Sends the same joint positions to the follower robot.
This is what makes the follower mimic the leader.

# Maintain 50Hz Loop Rate
```python
elapsed = time.time() - loop_start
sleep = DT - elapsed
if sleep > 0:
    time.sleep(sleep)

Measures how long the iteration took
Calculates how much time is left until 1/50th of a second passed
Sleeps the remaining time
Ensures the loop runs at a stable 50 Hz

# Shutdown
```python
print("[teleop] Done.")
follower.disconnect()
leader.disconnect()

Disconnects both robots cleanly.

# Script Entry Point
```python
if __name__ == "__main__":
    main()

Runs main() only if the script is executed directly.


"What This Script Achieves"
-connects the two robot arms
-Reads leader joint position
-sends those positions to the follower
-stops cleanly when you press Ctrl+C

# Teleoperation loop: leader->follower




# CAN-Transmission using IG Block [Vector CANoe]

##  Beginner CAN Transmission Project for ADAS Functions

![Screenshot 2025-05-18 115718](https://github.com/user-attachments/assets/16a2cbcb-7cc1-4bec-8f7e-9af97b7955d6)


This very simple project demonstrates the simulation and transmission of **6 essential ADAS (Advanced Driver Assistance Systems)** CAN messages. I built this to help beginners understand how different vehicle safety functions communicate over a CAN bus in a very simple way so that everybody who is just starting with learning communication protocols can refer to this example and draw real insights with examples.

Each message is associated with:

* A **CAN Identifier**
* A **Cycle Time**
* A **Data Length Code (DLC)**
* A **Payload**

---

###  Project Overview

| Function | CAN ID | DLC | Cycle Time | Purpose                                  |
| -------- | ------ | --- | ---------- | ---------------------------------------- |
| FCW      | 0x18A  | 8   | 50 ms      | Warns driver of imminent front collision |
| AEBS     | 0x1C4  | 8   | 20 ms      | Automatically brakes to avoid collision  |
| ACC      | 0x174  | 6   | 100 ms     | Maintains safe following distance        |
| LKA      | 0x1F2  | 8   | 50 ms      | Keeps vehicle centered in lane           |
| TSR      | 0x198  | 5   | 500 ms     | Detects traffic signs                    |
| BSD      | 0x1E5  | 4   | 100 ms     | Detects vehicles in blind spot           |

---

##  ADAS Functions and Their CAN Messages

![Screenshot 2025-05-18 120412](https://github.com/user-attachments/assets/c2594cb6-e9b7-48f4-bc15-ddcde04d3624)

---

### 1. **Forward Collision Warning (FCW)**

* **Purpose**: Warns the driver of an imminent frontal collision based on radar/camera input.
* **CAN ID**: `0x18A`
* **DLC**: `8`
* **Cycle Time**: `50 ms`
* **Payload**: `01 10 00 1E 05 00 00 7F`

**Reasoning:**

* **DLC** is 8 to accommodate info like obstacle type, TimeToCollision, speed, and counters.
* **Cycle Time** is 50 ms so the system can promptly update the warning if a vehicle is rapidly approaching. Lesser the better for emergency functions.
* **Shorter cycle time** improves driver reaction time and safety, this simply means the function is running at a rate of 50 times per second.
*  This also means that each sensor recording is collected at a rate of 50ms, also on the other side each sensor and actuator like RADAR or Camera and Brake ECU and ABS module are functioning at 50ms to respond.

---

### 2. **Autonomous Emergency Braking (AEBS)**

* **Purpose**: Automatically applies brakes if collision is imminent and driver doesn't react.
* **CAN ID**: `0x1C4`
* **DLC**: `8`
* **Cycle Time**: `20 ms`
* **Payload**: `02 01 00 0F 32 20 00 3C`

**Reasoning:**

* **DLC 8** allows sending deceleration command, object info, and TTC.
* **Very short cycle time (20 ms)** because **AEBS is safety-critical**; braking delay can be fatal.
* Faster cycle time means frequenet transmission of messages and faster sensing and actuations.
* Real-time updates are needed as vehicle dynamics change rapidly.
* This also means that there is heavy data collection and communication protocol should support such heavy transmission

---

### 3. **Adaptive Cruise Control (ACC)**

* **Purpose**: Automatically adjusts speed to maintain safe following distance.
* **CAN ID**: `0x174`
* **DLC**: `6`
* **Cycle Time**: `100 ms`
* **Payload**: `03 64 1E 02 00 21`

**Reasoning:**

* **DLC 6** is sufficient for speed, gap, and status because unlike AEBS, ACC is a comfort ADAS feature and not emergency feature.
* **Cycle Time is slower** at 100 ms since it's control-based, not immediate danger. Hence not only sensing but also actuations will be at 100ms.
* Adjustments are smooth and gradual on highways.

---

### 4. **Lane Keep Assist (LKA)**

* **Purpose**: Helps steer the vehicle to stay centered in the lane.
* **CAN ID**: `0x1F2`
* **DLC**: `8`
* **Cycle Time**: `50 ms`
* **Payload**: `04 01 64 80 00 0A 00 A0`

**Reasoning:**

* Needs 8 bytes to encode lane width, steering torque, and offsets.
* LKA fetches data mainly from front camera and also ORVM cameras to locate vehicles in the blind spot and to locate lane markings. 
* **Cycle time is 50 ms** to reflect lateral position changes in real time for effective steering correction.
* Since LKA assists in steering manoeuvre it is required to act swiftly when the vehicle is drifiting away from the lane especially at triple digit speeds.

---

### 5. **Traffic Sign Recognition (TSR)**

* **Purpose**: Detects and displays road signs like speed limits or no overtaking.
* **CAN ID**: `0x198`
* **DLC**: `5`
* **Cycle Time**: `500 ms`
* **Payload**: `05 31 00 03 2D`

**Reasoning:**

* **Small DLC (5)** since only sign type and confidence are sent.
* **Longest cycle time (500 ms)** as signs don’t change often.
* Updates are not safety-critical, so latency is acceptable. Hence 500ms

---

### 6. **Blind Spot Detection (BSD)**

* **Purpose**: Detects nearby vehicles in adjacent lanes.
* **CAN ID**: `0x1E5`
* **DLC**: `4`
* **Cycle Time**: `100 ms`
* **Payload**: `06 01 02 00`

**Reasoning:**

* **Small DLC (4)**—just left/right detection bits and status.
* **Moderate cycle time (100 ms)** is enough to track nearby vehicles. 
* Real-time feedback is necessary but not as critical as braking.

---

##  Why does Cycle Time Matters ?

![image](https://github.com/user-attachments/assets/d700a5df-36db-4fdd-bd22-51edf96e5973)


* **Shorter cycle time (e.g., 20–50 ms)** = **faster reaction** to dynamic, safety-critical scenarios like AEB or lane changes.
* **Longer cycle time (e.g., 100–500 ms)** is acceptable for functions like sign recognition or cruise control where immediate action is not required.
* **Too fast** = CAN bus overload, **too slow** = delayed response → system inefficiency or safety risk.

---

##  Why Do Some Functions Use Small DLC?

* Functions like **BSD** and **TSR** send only status flags or IDs → require fewer bytes.
* Using smaller DLC helps reduce CAN bus load and improves system efficiency.
* Critical systems like **AEBS** and **LKA** need more data → hence **full 8-byte DLC**.

---

# CAN-Transmission using Network Node [Vector CANoe]

![image](https://github.com/user-attachments/assets/704c5f25-5592-4833-a2fa-5dd78f54437b)


This project contains a CAPL script that implements **Advanced Driver Assistance Systems (ADAS)** signals using **SAE J1939** messages. The script defines message objects, dynamically transmits selected signals, and logs the simulation behavior.

---

##  Project Overview

- **Protocol**: SAE J1939 (29-bit identifier, used in commercial vehicles)
- **Tool**: Vector CANoe with CAPL scripting
- **Focus**: Simulating and tracing ADAS message behavior
- **Use Case**: Testing, validation, or training on J1939-based ADAS communication

---

##  CAPL Scripting Code & Trace Window

**CAPL Browser**

![image](https://github.com/user-attachments/assets/2216e9da-f7b0-4780-a87c-576b62803158)


**Trace Window**

![image](https://github.com/user-attachments/assets/c307c1fc-7abf-4eab-bd63-a64827457074)



---

##  List of ADAS Messages & Details

| #  | Message Name                     | PGN     | DLC | Data Bytes                 |
|----|----------------------------------|---------|-----|----------------------------|
| 1  | Lane Departure Warning           | 65265   | 8   | AA BB CC DD EE             |
| 2  | Adaptive Cruise Control Set Speed| 65217   | 4   | 10 20 30 40                |
| 3  | Collision Mitigation Braking     | 65266   | 6   | 15 25 35 45 55 65          |
| 4  | Blind Spot Detection             | 65269   | 3   | 5A 6B 7C                   |
| 5  | Automatic Emergency Braking      | 65267   | 7   | 40 50 60 70 80 90 A0       |
| 6  | Traffic Sign Recognition         | 65270   | 5   | 11 22 33 44 55             |
| 7  | Driver Drowsiness Detection      | 65271   | 8   | 77 88 99 AA BB CC DD EE    |
| 8  | Pedestrian Detection             | 65272   | 4   | 55 66 77 88                |
| 9  | Lane Keep Assist                 | 65273   | 6   | 31 42 53 64 75 86          |
| 10 | Headway Monitoring               | 65274   | 3   | 80 90 A0                   |
| 11 | Adaptive Headlight Control       | 65275   | 7   | 14 24 34 44 54 64 74       |
| 12 | Automatic Parking Assist         | 65276   | 5   | 20 30 40 50 60             |
| 13 | Vehicle Stability Control        | 65277   | 8   | B1 C2 D3 E4 F5 A6 B7 C8    |
| 14 | Rear Cross Traffic Alert         | 65278   | 4   | 10 21 32 43                |
| 15 | Road Condition Monitoring        | 65279   | 6   | 73 84 95 A6 B7 C8          |

---

##  Implementation Details

- **J1939 Protocol**: Widely adopted in commercial vehicles for robust, high-speed communication.
- **DLC Variation**: Each message uses a different DLC to match real-world signal structure.
- **CAPL Logic**: Messages are initialized and output using CAPL logic triggered at simulation start.

---

##  Usage Instructions

1. Import the CAPL script into your **Vector CANoe** environment.
2. Run the simulation.
3. Observe the ADAS message transmissions and signal behavior in the **Trace window**.
4. Modify or extend the message payloads for your use case.

---

##  Message Descriptions & DLC Breakdown

Each ADAS feature is designed with a specific DLC to reflect real data requirements:

### 1. **Lane Departure Warning** (PGN 65265, DLC = 8)
> Warns the driver when drifting from the lane.
- **Why 8?** Lane angle, deviation level, alert severity.

### 2. **Adaptive Cruise Control Set Speed** (PGN 65217, DLC = 4)
> Maintains target speed and adjusts based on traffic.
- **Why 4?** Speed, acceleration, mode status.

### 3. **Collision Mitigation Braking** (PGN 65266, DLC = 6)
> Applies brakes automatically to avoid collisions.
- **Why 6?** Distance to object, brake force, delay time.

### 4. **Blind Spot Detection** (PGN 65269, DLC = 3)
> Warns of vehicles in the blind spot.
- **Why 3?** Sensor state and object location.

### 5. **Automatic Emergency Braking** (PGN 65267, DLC = 7)
> Full braking in imminent crash scenarios.
- **Why 7?** Braking level, system status, proximity data.

### 6. **Traffic Sign Recognition** (PGN 65270, DLC = 5)
> Displays detected road signs.
- **Why 5?** Sign type, location, relevance.

### 7. **Driver Drowsiness Detection** (PGN 65271, DLC = 8)
> Detects driver fatigue.
- **Why 8?** Eye tracking, head nods, reaction time.

### 8. **Pedestrian Detection** (PGN 65272, DLC = 4)
> Detects pedestrians in vehicle path.
- **Why 4?** Position, urgency level.

### 9. **Lane Keep Assist** (PGN 65273, DLC = 6)
> Steers to keep vehicle centered.
- **Why 6?** Lane edges, torque, curvature.

### 10. **Headway Monitoring** (PGN 65274, DLC = 3)
> Maintains safe distance from the vehicle ahead.
- **Why 3?** Gap, time-to-collision.

### 11. **Adaptive Headlight Control** (PGN 65275, DLC = 7)
> Adjusts headlight brightness/direction.
- **Why 7?** Angle, brightness, adaptive mode.

### 12. **Automatic Parking Assist** (PGN 65276, DLC = 5)
> Helps vehicle park itself.
- **Why 5?** Slot size, steering plan.

### 13. **Vehicle Stability Control** (PGN 65277, DLC = 8)
> Maintains grip and traction.
- **Why 8?** Wheel slip, yaw, ABS status.

### 14. **Rear Cross Traffic Alert** (PGN 65278, DLC = 4)
> Detects side traffic while reversing.
- **Why 4?** Direction, threat level.

### 15. **Road Condition Monitoring** (PGN 65279, DLC = 6)
> Detects road grip and slipperiness.
- **Why 6?** Surface type, stability metrics.

---

## Final Summary

This project provides a clear beginner-friendly demonstration of transmitting real-world ADAS messages over CAN with accurate timing and data formatting. It introduces:

* CAN message structure
* Importance of DLC and cycle times
* How safety functions communicate efficiently

Note :  I will also write another article to explain how one can recreate this CAN transmission on VECTOR CANoe thus allowing anyone to start using CANoe with real examples.


---


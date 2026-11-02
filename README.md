# 🤖 4-DOF Pick and Place Autonomous Robot Car

![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![AVR](https://img.shields.io/badge/AVR-ATmega328P-red?style=for-the-badge)
![Robotics](https://img.shields.io/badge/Robotics-Mechatronics-orange?style=for-the-badge)

An autonomous line-following mobile robot equipped with a 4-DOF robotic manipulator and ultrasonic proximity targeting, engineered for automated material handling, trajectory tracking, and precision pick-and-place tasks.

🏆 **Awarded 1st Place out of 20 Teams** at the University EXPO Showcase in April 2024.

---

## 📌 Features

- **Autonomous Line Following Navigation**: Utilizes a dual IR reflectance sensor array to perform real-time closed-loop differential steering along ground track lines.
- **4-DOF Precision Robotic Manipulator**: Controlled via a 12-bit PCA9685 I2C PWM driver for multi-joint articulation (Base, Shoulder, Elbow, and End-Effector Gripper).
- **Ultrasonic Distance Ranging**: Integrates an HC-SR04 ultrasonic sensor to detect payload targets within a 22 cm threshold.
- **State-Driven Pick-and-Place Automation**: Executes timed, multi-stage kinematic sequences to lower the manipulator, actuate the gripper, hoist the object, and hold position during transport.
- **Optimized H-Bridge Differential Drive**: Directs dual DC motors with low-level Timer0 prescaler adjustment (`TCCR0B`) for smooth PWM motor speed control.

---

## 🧠 Kinematics & Control Logic

### 1. Differential Line Tracking Logic
The navigation system uses two analog IR sensors to control motor direction vectors:

- **Both IR LOW (On Track)**: Drives both left and right motors forward (`MOTOR_SPEED = 90`).
- **Right IR HIGH / Left IR LOW (Right Deviation)**: Reverses right motor and accelerates left motor (`-90, +90`) for pivot turning.
- **Right IR LOW / Left IR HIGH (Left Deviation)**: Accelerates right motor and reverses left motor (`+90, -90`) for pivot turning.
- **Both IR HIGH (Off Track / Stop)**: Halts both motors (`0, 0`).

### 2. PCA9685 PWM Servo Mapping
Joint angles ($0^\circ$ to $180^\circ$) are mapped to 12-bit pulse tick values ($150$ to $600$) at $60\text{ Hz}$ update frequency:

$$\text{Pulse Ticks} = \text{map}(\text{Angle}, 0, 180, 150, 600)$$

---

## 📐 System Architecture & Workflow

```text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                           SENSORS & INPUTS                                             │
├──────────────────────────────┬──────────────────────────────────────────┬──────────────────────────────┤
│ Dual IR Reflectance Array    │ HC-SR04 Ultrasonic Distance Sensor       │ PCA9685 I2C Servo Driver     │
│ (Analog Pins A0 & A1)        │ (Trigger A3 / Echo A2)                   │ (Address 0x40 @ 60Hz PWM)    │
└──────────────┬───────────────┴────────────────────┬─────────────────────┴──────────────┬───────────────┘
               │                                    │                                    │
               ▼                                    ▼                                    ▼
┌──────────────────────────────┐   ┌──────────────────────────────────┐   ┌──────────────────────────────┐
│ Differential Steering State  │   │ Object Proximity Evaluation      │   │ 4-DOF Joint Servo Control    │
│ (Line Follower Logic)        │   │ (Distance < 22 cm Threshold)     │   │ (Base, Shoulder, Elbow, Claw)│
└──────────────┬───────────────┘   └────────────────┬─────────────────┘   └──────────────┬───────────────┘
               │                                    │                                    │
               └─────────────────────────┬──────────┴────────────────────────────────────┘
                                         │
                                         ▼
                        ┌─────────────────────────────────┐
                        │     MAIN STATE MACHINE LOOP     │
                        ├─────────────────────────────────┤
                        │ • Line Navigation Active        │
                        │ • Automated Pick & Place State  │
                        │ • Post-Pickup Hold & Transport  │
                        └─────────────────────────────────┘
```

---

## 🛠️ Hardware Pinout & Wiring Configuration

| Component | Signal / Channel | Microcontroller Connection | Function |
|---|---|---|---|
| **Right Motor Driver** | Enable / IN1 / IN2 | `D10` (PWM) / `D9` / `D8` | Right wheel speed & direction |
| **Left Motor Driver** | Enable / IN1 / IN2 | `D11` (PWM) / `D7` / `D6` | Left wheel speed & direction |
| **Right IR Sensor** | Analog Data | `A0` | Line detection input (Right) |
| **Left IR Sensor** | Analog Data | `A1` | Line detection input (Left) |
| **Ultrasonic Sensor** | Echo / Trigger | `A2` (Echo) / `A3` (Trigger) | Target distance measurement |
| **PCA9685 Servo Driver** | Hardware I2C | `SDA` (`A4`) / `SCL` (`A5`) | 4-DOF Arm PWM Control (0x40) |
| **Robotic Arm Servos** | Channels 0, 1, 2, 3 | PCA9685 PWM Header Pins | Base (Ch0), Shoulder (Ch1), Elbow (Ch2), Claw (Ch3) |

---

## ⚙️ Tech Stack & Dependencies

- **Language:** C++ / Native C (Arduino Framework)
- **Microcontroller Core:** ATmega328P (Arduino Uno)
- **Libraries:** `Wire.h` (I2C), `Adafruit_PWMServoDriver.h`
- **Control Hardware:** L298N / L293D H-Bridge Motor Driver, PCA9685 16-Channel 12-Bit PWM Driver

---

## 🚀 How to Run the System

### 1. Hardware Assembly
1. Connect the dual H-bridge motor driver inputs and enable pins to Arduino digital pins `D6` through `D11`.
2. Connect the IR line sensors to analog pins `A0` and `A1`.
3. Wire the HC-SR04 ultrasonic sensor to pins `A2` (Echo) and `A3` (Trigger).
4. Connect the PCA9685 PWM driver to the Arduino I2C bus (`SDA`/`SCL`).
5. Plug the 4 robotic arm servos into PWM channels 0, 1, 2, and 3 on the PCA9685 module.

### 2. Software Flashing
1. Install the **Adafruit PWM Servo Driver Library** in the Arduino IDE.
2. Open `4DOF_Pick_and_Place_Robot_Car.ino`.
3. Select **Arduino Uno** as the target board, choose the serial port, and click **Upload**.

---

## 📂 Project Structure

```text
4DOF_Pick_and_Place_Robot_Car/
│
├── 4DOF_Pick_and_Place_Robot_Car.ino  # Main control software
└── README.md                          # Project documentation
```

---

## 📜 Certification & Context

Designed and fabricated between **January 2024 and April 2024**. The project **ranked 1st out of 20 competing teams** at the University EXPO Showcase in April 2024.

---

## 👨‍💻 Author

**Mahmoud El-Adgham**  
Electrical & Electronic Engineering | AI & Machine Learning  

* **GitHub:** [El-adghamF90](https://github.com/El-adghamF90)
* **LinkedIn:** [Mahmoud EL-Adgham](https://www.linkedin.com/in/mahmoud-el-adgham/)

# 🤖 Robotic Nurse Assistant
> Bare-metal ARM Assembly firmware on STM32F401RCT6 — combining mobility, medical assistance, and embedded control into a single robotic platform.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Features](#-features)
- [Hardware Components](#-hardware-components)
- [System Architecture](#-system-architecture)
- [Communication Protocols](#-communication-protocols)
- [Software Architecture](#-software-architecture)
- [How the Robot Works](#-how-the-robot-works)
- [Motor Control Logic](#-motor-control-logic)
- [Interrupt Handling](#-interrupt-handling)
- [GSM SMS Flow](#-gsm-sms-flow)
- [LCD Interface](#-lcd-interface)
- [Assembly Code Structure](#-assembly-code-structure)
- [Pin Connections](#-pin-connections)
- [Setup Instructions](#-setup-instructions)
- [Challenges Faced](#-challenges-faced)
- [Project Gallery](#project-gallery)
- [Contributors](#-contributors)

---

## 📌 Project-Overview

The **Robotic Nurse Assistant** is a microprocessor course project built around the **STM32F401RCT6** ARM Cortex-M4 microcontroller. The entire firmware was written in **ARM Assembly**.

The robot is designed to assist in healthcare environments by combining mobility, patient communication, temperature monitoring, automatic sanitization, medicine dispensing, and a robotic arm into a single integrated platform.

| Property | Details |
|---|---|
| **Microcontroller** | STM32F401RCT6 |
| **Architecture** | ARM Cortex-M4 |
| **Language** | ARM Assembly (bare-metal) |
| **Development Duration** | ~2 months |
| **Team Size** | 10 members |
| **Prototype Type** | Breadboard / module-based |

---

## ✨ Features

### 🚗 Mobility System
- 6-wheel differential drive platform (3 wheels per side)
- Remote-controlled via standard **TV IR remote** and IR receiver
- Supports: **Forward · Backward · Left · Right · Stop**

### 📡 GSM Communication
- **SIM800L** module via UART
- Sends emergency SMS alerts and patient request messages to nursing staff

### 🌡️ Temperature Monitoring
- **LM35DZ** analog sensor read through ADC
- Temperature displayed in real-time on the LCD

### 🧴 Automatic Alcohol Sanitizer
- IR sensor triggers an interrupt on hand detection
- Automatically activates the spray mechanism — no button press needed

### 💊 Medicine Dispensing System
- Rotating drawer controlled by a **stepper motor**
- Multiple medicine compartments selectable by rotation

### 🦾 Robotic Arm
- Fixed sequence: move down → pick up medicine → lift
- Includes injection-assistance functionality

### 🖥️ Display System
- LCD connected via **I2C**
- Shows temperature readings, system status, and notifications

### 🔊 Audio Feedback
- **DFPlayer Mini** module with SD card
- Plays pre-recorded audio notifications and alerts

---

## 🔩 Hardware Components

| Component | Model / Type | Purpose |
|---|---|---|
| Microcontroller | STM32F401RCT6 | Main controller |
| Robot Chassis | 6-wheel platform | Mobility |
| DC Motors | Standard DC | Drive wheels |
| IR Remote + Receiver | Standard TV IR | Remote control |
| GSM Module | SIM800L | SMS communication |
| Temperature Sensor | LM35DZ | Analog temp sensing |
| Display | 16x2 or 20x4 LCD | I2C display |
| Audio Module | DFPlayer Mini | Sound notifications |
| Stepper Motor | Standard stepper | Medicine drawer |
| Servo / DC Motors | — | Robotic arm joints |
| IR Sensor | Proximity IR | Sanitizer trigger |
| SD Card | — | Audio files storage |
| Power Supply | — | System power |

---

## 🏗️ System Architecture

```
                         ┌─────────────────────────────────┐
                         │        STM32F401RCT6             │
                         │        ARM Cortex-M4             │
                         │      (ARM Assembly Firmware)     │
                         └──────────────┬──────────────────┘
                                        │
          ┌─────────────────────────────┼─────────────────────────────┐
          │                             │                             │
   ┌──────▼──────┐              ┌───────▼──────┐             ┌───────▼──────┐
   │   GPIO/PWM  │              │     UART     │             │    I2C Bus   │
   └──────┬──────┘              └───────┬──────┘             └───────┬──────┘
          │                             │                             │
   ┌──────┴──────────────┐      ┌───────┴──────┐             ┌───────┴──────┐
   │                     │      │   SIM800L    │             │     LCD      │
   │  ┌──────────────┐   │      │  GSM Module  │             │   Display    │
   │  │  DC Motors   │   │      └──────────────┘             └──────────────┘
   │  │ (6-wheel     │   │
   │  │  drive)      │   │      ┌──────────────┐
   │  └──────────────┘   │      │  DFPlayer    │◄── SD Card (Audio Files)
   │                     │      │  (UART/GPIO) │
   │  ┌──────────────┐   │      └──────────────┘
   │  │  Stepper     │   │
   │  │  Motor       │   │      ┌──────────────┐
   │  │ (Medicine    │   │      │   ADC Input  │
   │  │  Drawer)     │   │      └──────┬───────┘
   │  └──────────────┘   │             │
   │                     │      ┌──────▼───────┐
   │  ┌──────────────┐   │      │    LM35DZ    │
   │  │  Robotic Arm │   │      │  Temp Sensor │
   │  │  (Servo/DC)  │   │      └──────────────┘
   │  └──────────────┘   │
   │                     │      ┌──────────────┐
   │  ┌──────────────┐   │      │   EXTI/IRQ   │
   │  │  IR Receiver │   │      └──────┬───────┘
   │  │ (TV Remote)  │   │             │
   │  └──────────────┘   │      ┌──────▼───────┐
   │                     │      │  IR Sensor   │
   │  ┌──────────────┐   │      │ (Sanitizer   │
   │  │  Sanitizer   │◄──┘      │  Trigger)    │
   │  │  Pump/Spray  │          └──────────────┘
   │  └──────────────┘
   └─────────────────────┘
```

---

## 📶 Communication Protocols

### UART
Used for serial communication with:
- **SIM800L** GSM module — AT command-based SMS sending
- **DFPlayer Mini** — audio playback control

The UART peripheral is configured at the register level: baud rate divisor calculated manually and loaded into `USART_BRR`, with TX/RX pins set to alternate function mode via `GPIOx_MODER` and `GPIOx_AFR`.

### I2C
Used for:
- **LCD Display** — sending character data and control commands through the I2C-to-parallel backpack (typically PCF8574)

I2C is initialized by configuring `I2C_CR1`, `I2C_CR2`, and `I2C_CCR` registers directly. Start condition, address byte, data byte, and stop condition are all bit-banged at the register level.

### ADC
Used for:
- **LM35DZ** temperature sensor — 10mV/°C analog output converted to digital

The ADC is configured in single-conversion mode. The raw 12-bit result from `ADC_DR` is read and converted to Celsius using fixed-point arithmetic in Assembly.

---

## 🧠 Software Architecture

The firmware is structured as a set of modular Assembly routines, each handling a specific subsystem. There is no operating system — all control flow is managed through the main loop, interrupt service routines (ISRs), and subroutine calls.

```
Robotic-Nurse-Ziko/
│
├── Main.s               ; System init, peripheral setup, and main control loop
├── Movement.s           ; DC motor control, differential drive, IR remote decoding
├── Arm.s                ; Robotic arm fixed movement sequence (down → grab → lift)
├── Syringe.s            ; Injection-assistance functionality and arm extension
├── Drawer.s             ; Stepper motor control for rotating medicine drawer
├── Sprayer.s            ; IR sensor interrupt handling and sanitizer pump activation
├── Gsm.s                ; SIM800L UART communication and AT command SMS sequences
├── Display.s            ; I2C LCD initialization, character/string write routines
└── Audio.s              ; DFPlayer Mini control for audio notifications and alerts
```

---

## ⚙️ How the Robot Works

### Startup Sequence
1. System clock configured (HSI/PLL) via `RCC` registers
2. All peripheral clocks enabled in `RCC_AHB1ENR` / `RCC_APB1ENR` / `RCC_APB2ENR`
3. GPIO pins configured for each subsystem
4. UART, I2C, ADC, Timers initialized
5. NVIC interrupt priorities set
6. LCD displays boot message
7. DFPlayer plays startup audio
8. Main loop begins

### Main Loop
```
LOOP:
  ├── Poll IR receiver → decode command → update motor outputs
  ├── Read ADC (LM35DZ) → convert to °C → update LCD
  ├── Check GSM flags → send SMS if triggered
  ├── Check arm/dispenser flags → execute movement if triggered
  └── BL LOOP
```

### Interrupt-Driven Events
- **Sanitizer IR sensor** → EXTI line → triggers pump ISR immediately
- **Timer interrupts** → stepper motor step timing

---

## 🚗 Motor Control Logic

### DC Motors (Differential Drive)

The 6-wheel platform uses two independent motor channels. Two wheels on each side share the same output line, making it effectively a 4-output system.

| Command | Left Motors | Right Motors |
|---|---|---|
| Forward | Forward | Forward |
| Backward | Backward | Backward |
| Turn Left | Backward / Stop | Forward |
| Turn Right | Forward | Backward / Stop |
| Stop | Stop | Stop |

Motor direction is controlled via **GPIO output pins** to an H-bridge driver. Speed is optionally controlled via **PWM** (Timer output compare mode), configured through `TIMx_CCR` and `TIMx_CCER` registers.

### Stepper Motor (Medicine Drawer)

The stepper motor uses a **4-step full-step sequence** driven by 4 GPIO output pins:

```
Step 1: 1000
Step 2: 0100
Step 3: 0010
Step 4: 0001
```

Each step is separated by a software delay loop. The number of steps determines how far the drawer rotates.

---

## 🔔 Interrupt Handling

### External Interrupt — Sanitizer (EXTI)
- IR sensor connected to a GPIO pin configured as EXTI source
- `SYSCFG_EXTICRx` maps the pin to the EXTI line
- Rising/falling edge detection configured in `EXTI_RTSR` / `EXTI_FTSR`
- ISR clears the pending flag in `EXTI_PR` and activates the pump output

### Timer Interrupts
- Used for stepper motor step timing and PWM generation
- `TIMx_DIER` enables update interrupt
- `TIMx_ARR` sets the period
- ISR advances the stepper sequence state machine

### NVIC Configuration
All interrupt priorities and enables are configured directly through the NVIC registers (`NVIC_IPRx`, `NVIC_ISERx`) in Assembly.

---

## 📨 GSM SMS Flow

The SIM800L module is controlled via **AT commands** sent over UART.

```
1. Send: AT\r\n                  → Wait for OK
2. Send: AT+CMGF=1\r\n           → Set SMS text mode → Wait for OK
3. Send: AT+CMGS="+XXXXXXXXXXX"\r\n → Set recipient number
4. Wait for '>' prompt
5. Send: <message text>
6. Send: Ctrl+Z (0x1A)           → Finalize and send SMS
7. Wait for +CMGS confirmation
```

Two message types are supported:
- **Emergency Alert** — critical patient condition notification
- **Patient Request** — patient needs assistance message

---

## 🖥️ LCD Interface

The LCD is connected via an **I2C backpack** (PCF8574 I/O expander). All communication goes through the I2C bus at the register level.

### Write Sequence
```
1. I2C Start condition
2. Send device address (0x27 or 0x3F) + Write bit
3. Send 4-bit nibble with Enable=1 (latch high)
4. Send 4-bit nibble with Enable=0 (latch low)
5. Repeat for second nibble (LCD uses 4-bit mode)
6. I2C Stop condition
```

### Displayed Information
- Line 1: Real-time temperature in °C
- Line 2: System status / active mode / notifications

---

## 🔌 Pin Connections

| Pin  | Connected To            | Purpose                           |
|------|-------------------------|-----------------------------------|
| PA0  | IR Receiver             | TV remote input (EXTI0 interrupt) |
| PA2  | USART2 TX / DFPlayer    | Audio module communication        |
| PA3  | USART2 RX / DFPlayer    | Audio module communication        |
| PA4  | LM35DZ                  | Temperature sensor analog input   |
| PA5  | Motor driver            | Movement control                  |
| PA6  | Motor driver            | Movement control                  |
| PA7  | Motor driver            | Movement control                  |
| PA8  | Motor driver            | Movement control                  |
| PA9  | SIM800L TX (USART1_TX)  | GSM communication                 |
| PA10 | SIM800L RX (USART1_RX)  | GSM communication                 |
| PA11 | PWM output (TIM1 CH4)   | Alcohol sprayer actuator          |
| PA12 | IR Sensor               | Alcohol spray trigger interrupt   |
| PA15 | Stepper motor           | Medicine drawer control           |
| PB0  | Motor driver            | Movement control                  |
| PB1  | Motor driver            | Movement control                  |
| PB2  | Motor driver            | Movement control                  |
| PB3  | Stepper motor           | Medicine drawer control           |
| PB4  | Stepper motor           | Medicine drawer control           |
| PB5  | Stepper motor           | Medicine drawer control           |
| PB6  | I2C LCD SCL             | LCD clock                         |
| PB7  | I2C LCD SDA             | LCD data                          |
| PB8  | Arm motor/servo control | Robotic arm                       |
| PB9  | Arm motor/servo control | Robotic arm                       |
| PB10 | Motor driver            | Movement control                  |
| PB12 | Arm motor/servo control | Robotic arm                       |
| PB13 | Arm motor/servo control | Robotic arm                       |
| PB14 | Arm motor/servo control | Robotic arm                       |
| PB15 | Arm motor/servo control | Robotic arm                       |
| PC13 | GPIO output             | General output                    |
| PC14 | GPIO output             | General output                    |
| PC15 | GPIO output             | General output                    |

---

## 🛠️ Setup Instructions

### Prerequisites
- **STM32CubeIDE** or any ARM GCC toolchain with Assembly support
- **ST-Link V2** programmer/debugger
- **STM32CubeProgrammer** for flashing
- SIM card installed in SIM800L with SMS capability
- Audio files loaded onto DFPlayer SD card
- Hardware assembled per the wiring diagram

### Build & Flash

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-org/robotic-nurse-assistant.git
   cd robotic-nurse-assistant
   ```

2. **Open project in STM32CubeIDE**
   - Import as existing project
   - Confirm the target is set to STM32F401RCT6

3. **Build the firmware**
   ```
   Project → Build All  (or Ctrl+B)
   ```

4. **Flash to the board**
   - Connect ST-Link to the STM32 SWD header (SWDIO, SWCLK, GND, 3.3V)
   - Run → Debug (or use STM32CubeProgrammer to flash the `.bin` / `.elf`)

5. **Power the system**
   - Ensure motor driver and GSM module have adequate power supply
   - GSM module requires up to 2A peak — use a dedicated supply

6. **Test**
   - Point IR remote at the receiver and test movement commands
   - Wave a hand near the sanitizer IR sensor to test auto-spray
   - Monitor UART output for GSM AT command responses

---

## ⚡ Challenges Faced

| Challenge | How We Addressed It |
|---|---|
| Writing all firmware in ARM Assembly | Carefully studied the STM32F401 reference manual and wrote modular subroutines for each peripheral |
| GSM AT command timing | Added polling loops to wait for specific response strings byte-by-byte over UART |
| I2C LCD bit-level control | Manually implemented the PCF8574 nibble-write protocol in Assembly |
| IR remote signal decoding | Measured pulse widths using timer capture and matched against NEC protocol timings |
| Stepper motor jitter | Tuned the delay loop timing between steps to ensure smooth rotation |
| Multi-module hardware integration | Careful power distribution and systematic debugging of one module at a time |
| ADC temperature accuracy | Applied fixed-point scaling to convert 12-bit ADC result to accurate °C values |

---

## Project-Gallery

### 📸 Photos

| | |
|---|---|
| ![Robot Front View](images/robot_front.jpeg) | ![LCD Display](images/lcd_display.jpeg) |
| Robot Front View | LCD Temperature Display |

### 🎥 Demo Videos

| Video | Description |
|---|---|
| [📱 SMS Testing](https://drive.google.com/file/d/1ZhcokTljZK90xDJjAGgwmMymoWi-Lifq/view?usp=drive_link) | SIM800L sending emergency and patient request SMS alerts via USART1 AT commands |
| [🔔 Robot Ring / Audio](https://drive.google.com/file/d/1uhTMvsidw-IoVPXe2Mzyv1zYMgSA-ZNL/view?usp=drive_link) | DFPlayer audio notification system playing pre-recorded alerts |
| [🔄 360° View & Movement](https://drive.google.com/file/d/11GONHEGwSxaOF7qzNA99Y5vbGS1wmrRp/view?usp=drive_link) | Full robot overview with live differential drive movement demo — forward, backward, left, right |
| [🦾 Robotic Arm](https://drive.google.com/file/d/11vmQge6gEoCyOmKDpHcNmTbRrLkG8fAQ/view?usp=drive_link) | Fixed arm sequence: move down → pick up medicine → lift |

---

## 👥 Contributors

| Name | Contributions |
|---|---|
| Ahmed Abd Alaziz | Robot chassis design, alcohol sanitizer system |
| Ahmed Nasser | Rotating medicine drawer, LCD interface |
| Adam Khaled | Robotic arm design & control |
| Adam Ehab |  buzzer integration |
| Adham Ashraf | GSM module, audio system, robotic arm mechanical design |
| Abdullah Hussein | Injection assistance system |
| Abdullah Hesham | Temperature sensor, hardware integration, robotic arm |
| Abdelrahman Eid | Motion / movement control system |
| Omar Moharam | Injection assistance system |
| Yousef Hesham | IR remote control system |

---

## 📄 License

This project was developed as part of a university microprocessors course. All rights reserved by the project team.

---

<div align="center">
  <strong>Built with ❤️ in ARM Assembly — no shortcuts, no HAL, no mercy.</strong>
  <br/>
  <sub>STM32F401RCT6 · ARM Cortex-M4 · Bare-Metal Embedded Systems</sub>
</div>

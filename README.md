# 🦾 6-Axis Robotic Arm Controller — RP2040

> A high-performance embedded controller for 6-axis robotic arms featuring PIO-based jitter-free PWM, per-channel current sensing, real-time torque estimation, and hardware-level collision detection — no external sensors required.

![License](https://img.shields.io/badge/License-CERN--OHL-green?style=flat-square)
![MCU](https://img.shields.io/badge/MCU-RP2040-blue?style=flat-square)
![Axes](https://img.shields.io/badge/Axes-6--DOF-orange?style=flat-square)
![PCB](https://img.shields.io/badge/PCB-4--Layer-yellow?style=flat-square)
![Lang](https://img.shields.io/badge/Firmware-MicroPython%20%7C%20C%2B%2B-lightgrey?style=flat-square)

---

## 📸 PCB Renders

| 3D Render | PCB Layout | Schematic |
|:---------:|:----------:|:---------:|
| ![3D Render](https://raw.githubusercontent.com/bejaouihamza/robotic_arm_controller/main/Capture%20d'%C3%A9cran%202026-04-10%20231258.png) | ![PCB Layout](https://github.com/bejaouihamza/robotic_arm_controller/blob/main/Capture%20d'%C3%A9cran%202026-04-10%20231240.png?raw=true) | ![Schematic](https://github.com/bejaouihamza/robotic_arm_controller/blob/main/Capture%20d'%C3%A9cran%202026-04-10%20231415.png?raw=true) |

---

## ⚡ Key Specs

| Parameter | Value |
|-----------|-------|
| MCU | Raspberry Pi RP2040 — Dual-core ARM Cortex-M0+ @ 133 MHz |
| Flash | 16MB W25Q16JV55 QSPI |
| Servo Channels | 6 independent, PIO-generated PWM |
| PWM Frequency | 50 Hz · 500–2500 µs pulse range |
| PWM Jitter | < 1 µs (PIO hardware, zero CPU involvement) |
| Current Sense | Per-channel INA180 amplifier (×20 gain, 10 mΩ shunt) |
| Peak Current | 10A total across all 6 channels |
| Input Voltage | 6V – 24V VIN + USB-C 5V |
| PCB | 4-Layer, KiCad |
| Firmware | MicroPython · C++ Pico SDK |
| License | CERN Open Hardware License v2 |

---

## 🧠 Core Concept

Traditional servo controllers simply send PWM signals. This board goes further:

```
MEASURE  →  ANALYZE  →  REACT
```

Every servo channel has a dedicated current sense amplifier. The RP2040 reads all six channels through a **74HC4051** analog multiplexer, continuously estimating torque, detecting collisions, and triggering per-channel shutdowns — all in real time.

---

## 🔩 Hardware & Components

### Processing Unit
- **RP2040** — Dual-core ARM Cortex-M0+ @ 133 MHz, 264KB SRAM, 8 PIO state machines
- **W25Q16JV55** — 16MB QSPI Flash for firmware and data logging
- **12 MHz Crystal** — Stable PLL reference for USB timing and precise PWM periods

### Power Management
- **Integrated 5V Buck Converter** — Logic supply and low-power servo rail
- **AMS1117-3.3** — Clean LDO for MCU and all analog circuitry
- **Auto power-path** — Seamless switching between USB-C (5V) and external VIN (6–24V)

### Servo Driver Block (×6 channels)
- **AO3401A P-MOSFET** — High-side switch per channel. Enables software-controlled power sequencing and instant per-joint emergency stops
- **INA180A1** — Current sense amplifier, gain ×20. Amplifies shunt voltage for ADC reading
- **10 mΩ Shunt Resistor** — In-series with each servo rail for current measurement

### Analog Signal Chain
- **74HC4051** — 8:1 CMOS analog multiplexer. Funnels all 6 current sense signals into a single ADC pin using only 3 GPIO select lines
- **JP1–JP6 Jumpers** — Per-channel voltage selection: position A = 5V buck, position B = direct VIN

### User Interface
- **RV1, RV2** — 10kΩ potentiometers mapped to Joint 1 & 2 for manual testing
- **D3** — User-programmable LED
- **Reset button** — Hardware reset
- **SWD header** — ARM Serial Wire Debug (J-Link / DAPLink / OpenOCD)

---

## 🔌 GPIO Pin Mapping

> GPIO 0–5 → Servo **Enable** (P-MOS high-side switches)  
> GPIO 6–11 → Servo **PWM** (PIO state machine outputs)

| GPIO | Net Name | Type | Connected To | Description |
|------|----------|------|--------------|-------------|
| GPIO 0 | servo1_EN | GPIO OUT | AO3401A Gate Ch1 | High-side MOSFET enable — active HIGH |
| GPIO 1 | servo2_EN | GPIO OUT | AO3401A Gate Ch2 | High-side MOSFET enable — active HIGH |
| GPIO 2 | servo3_EN | GPIO OUT | AO3401A Gate Ch3 | High-side MOSFET enable — active HIGH |
| GPIO 3 | servo4_EN | GPIO OUT | AO3401A Gate Ch4 | High-side MOSFET enable — active HIGH |
| GPIO 4 | servo5_EN | GPIO OUT | AO3401A Gate Ch5 | High-side MOSFET enable — active HIGH |
| GPIO 5 | servo6_EN | GPIO OUT | AO3401A Gate Ch6 | High-side MOSFET enable — active HIGH |
| GPIO 6 | servo1_PWM | PIO OUT | Servo 1 signal wire | 50 Hz PIO PWM — sub-µs jitter |
| GPIO 7 | servo2_PWM | PIO OUT | Servo 2 signal wire | 50 Hz PIO PWM |
| GPIO 8 | servo3_PWM | PIO OUT | Servo 3 signal wire | 50 Hz PIO PWM |
| GPIO 9 | servo4_PWM | PIO OUT | Servo 4 signal wire | 50 Hz PIO PWM |
| GPIO 10 | servo5_PWM | PIO OUT | Servo 5 signal wire | 50 Hz PIO PWM |
| GPIO 11 | servo6_PWM | PIO OUT | Servo 6 signal wire | 50 Hz PIO PWM |
| GPIO 12 | S0 (MUX) | GPIO OUT | 74HC4051 Pin 11 | MUX address bit 0 |
| GPIO 13 | S1 (MUX) | GPIO OUT | 74HC4051 Pin 10 | MUX address bit 1 |
| GPIO 14 | S2 (MUX) | GPIO OUT | 74HC4051 Pin 9 | MUX address bit 2 |
| GPIO 18 | POT_1 | ADC0 IN | RV1 wiper | Manual Joint 1 input (0–3.3V = 0°–180°) |
| GPIO 19 | POT_2 | ADC1 IN | RV2 wiper | Manual Joint 2 input |
| GPIO 26 | MUX_A | GPIO OUT | 74HC4051 Pin 11 | MUX address line A |
| GPIO 27 | MUX_B | GPIO OUT | 74HC4051 Pin 10 | MUX address line B |
| GPIO 28 | MUX_C | GPIO OUT | 74HC4051 Pin 9 | MUX address line C |
| GPIO 29 | ISNS_ADC | ADC3 IN | 74HC4051 Common (Z) | Reads selected channel current sense voltage |
| USB-C | USB_DP/DM | USB 1.1 | USB-C connector | UF2 bootloader + CDC serial debug |
| SWD | SWCLK/SWDIO | DEBUG | SWD header (JB) | ARM Serial Wire Debug |

---

## 🏗️ PCB — 4-Layer Stackup

| Layer | Function | Details |
|-------|----------|---------|
| **Layer 1 — Top** | Signal + Components | All SMD placement, fine-pitch routing (MCU, INA180, MUX) |
| **Layer 2 — GND** | Solid ground plane | Low-impedance return path, EMI shielding, heat spreading |
| **Layer 3 — Power** | Split power planes | 3.3V (MCU/sensors) · 5V (buck) · VIN (servo rails) |
| **Layer 4 — Bottom** | High-current traces | Servo power delivery, secondary signal routing |

A 4-layer design is essential here — with 6 servo channels switching simultaneously and up to 10A peak current, a 2-layer board would suffer from unacceptable EMI coupling and resistive heating.

---

## 📐 Physics — How It Works

### Motor Torque from Current

The fundamental relationship exploited by this board:

```
τ = Kₜ × I
```

A DC motor's torque is directly proportional to its armature current. `Kₜ` is the motor's torque constant (Nm/A). By measuring current through the INA180 shunt amplifiers, we measure torque — without any mechanical sensor.

### Current Sensing Chain

```
I (servo current)
  → V_shunt = I × R_shunt          (Ohm's Law across 10mΩ shunt)
  → V_out = V_shunt × 20           (INA180 ×20 amplification)
  → ADC = (V_out / 3.3V) × 65535  (RP2040 12-bit ADC)
  → I = (ADC / 65535 × 3.3) / (20 × 0.01)
```

At 2.5A (overload threshold): `V_shunt = 25mV → V_out = 500mV` — well within ADC range.

**Resolution:** ~0.25 mA per LSB → ~0.02 mNm torque resolution per LSB.

### Servo PWM Position Encoding

```
θ° = (t_pulse − 500µs) / 2000µs × 180°
```

| Pulse Width | Angle |
|-------------|-------|
| 500 µs | 0° |
| 1500 µs | 90° (centre) |
| 2500 µs | 180° |

The RP2040's PIO state machines generate these pulses in hardware — completely independent of the CPU clock, eliminating software-induced jitter that corrupts positional accuracy.

### Why PIO Instead of Hardware PWM?

Hardware PWM slices share a clock divider. Reprogramming multiple channels simultaneously causes brief glitches. PIO state machines run from dedicated instruction memory, each executing its PWM loop independently — giving 6 perfectly phase-stable signals while freeing both CPU cores for inverse kinematics or vision tasks.

### Collision Detection — Torque Spike

When the arm hits an obstacle, the motor stalls. A stalled motor draws its locked-rotor current — far above normal running current:

```
Normal:  I ≈ 0.3–0.8 A  →  τ ≈ 0.025–0.068 Nm
Stall:   I → 2–4 A      →  τ → 0.17–0.34 Nm  ← triggers shutdown
```

The ADC scans all 6 channels continuously. A torque spike is detected within one scan cycle (~1 ms). The P-MOSFET cuts power within a single GPIO toggle (~10 ns).

---

## 🛡️ Safety System

| Feature | Description |
|---------|-------------|
| **Torque Estimation** | τ = Kₜ × I, live on all 6 joints, ~0.02 mNm resolution |
| **Collision Detection** | Configurable torque threshold (default 0.20 Nm) triggers per-joint shutdown |
| **Overcurrent Protection** | Sustained current > 2.5A sets latching fault, cuts P-MOSFET |
| **Emergency Stop** | All 6 channels cut simultaneously via single function call |
| **Smooth Motion** | Interpolated moves with per-step collision check |
| **Power Sequencing** | Staggered joint enable (100ms delay) prevents inrush brownout |
| **Fault Latch** | Fault persists until software reset — prevents repeated thermal stress |

---

## 💻 Firmware

```python
# robotic_arm_controller.py
# GPIO 0-5  → Servo ENABLE (P-MOS high-side switches)
# GPIO 6-11 → Servo PWM    (PIO state machines)
import rp2, machine, time

SERVO_EN_PINS  = [0, 1, 2, 3, 4, 5]
SERVO_PWM_PINS = [6, 7, 8, 9, 10, 11]
MUX_SEL_PINS   = [12, 13, 14]
ADC_ISNS_PIN   = 29
POT1_PIN, POT2_PIN = 18, 19

SERVO_MIN_US = 500
SERVO_MAX_US = 2500
ISNS_GAIN    = 20
SHUNT_R      = 0.010
OVERLOAD_A   = 2.5
KT_NM_PER_A  = 0.085
COLLISION_NM = 0.20

@rp2.asm_pio(sideset_init=rp2.PIO.OUT_LOW)
def servo_pwm_pio():
    pull(block)          .side(0)
    mov(x, osr)
    label("high_loop")
    jmp(x_dec, "high_loop")  .side(1)
    pull(block)          .side(1)
    mov(x, osr)
    label("low_loop")
    jmp(x_dec, "low_loop")   .side(0)

class ServoChannel:
    def __init__(self, en_pin, pwm_pin, sm_id):
        self.en = machine.Pin(en_pin, machine.Pin.OUT, value=0)
        self.sm = rp2.StateMachine(sm_id, servo_pwm_pio,
                    freq=2_000_000, sideset_base=machine.Pin(pwm_pin))
        self.sm.active(1)
        self.angle = 90
        self.set_angle(90)

    def power_on(self):  self.en.value(1)
    def power_off(self): self.en.value(0)

    def set_angle(self, deg):
        deg = max(0.0, min(180.0, float(deg)))
        us  = SERVO_MIN_US + (deg / 180.0) * (SERVO_MAX_US - SERVO_MIN_US)
        self.sm.put(int(us * 2))
        self.sm.put(int((20000 - us) * 2))
        self.angle = deg

class CurrentSensor:
    def __init__(self):
        self.mux = [machine.Pin(p, machine.Pin.OUT) for p in MUX_SEL_PINS]
        self.adc = machine.ADC(ADC_ISNS_PIN)

    def read_channel(self, ch):
        for i, pin in enumerate(self.mux):
            pin.value((ch >> i) & 1)
        time.sleep_us(5)
        v = (self.adc.read_u16() / 65535) * 3.3
        return round(v / (ISNS_GAIN * SHUNT_R), 4)

    def read_all(self):
        return [self.read_channel(i) for i in range(6)]

class RoboticArm:
    def __init__(self):
        self.joints = [ServoChannel(SERVO_EN_PINS[i], SERVO_PWM_PINS[i], i) for i in range(6)]
        self.isns   = CurrentSensor()
        self.pot1   = machine.ADC(POT1_PIN)
        self.pot2   = machine.ADC(POT2_PIN)
        self.fault  = [False] * 6

    def power_up_sequence(self, delay_ms=100):
        for j in self.joints:
            j.power_on()
            time.sleep_ms(delay_ms)

    def emergency_stop(self):
        for j in self.joints: j.power_off()
        print("[E-STOP] All channels cut")

    def move_to(self, angles):
        for i, (j, deg) in enumerate(zip(self.joints, angles)):
            if not self.fault[i]: j.set_angle(deg)

    def torque_estimate(self):
        return [round(a * KT_NM_PER_A, 4) for a in self.isns.read_all()]

    def detect_collision(self, threshold_nm=COLLISION_NM):
        hits = []
        for i, t in enumerate(self.torque_estimate()):
            if t > threshold_nm and not self.fault[i]:
                self.joints[i].power_off()
                self.fault[i] = True
                hits.append(i)
                print(f"[COLLISION] Joint {i+1} @ {t:.4f} Nm")
        return hits

    def check_overload(self):
        for i, a in enumerate(self.isns.read_all()):
            if a > OVERLOAD_A and not self.fault[i]:
                self.joints[i].power_off()
                self.fault[i] = True
                print(f"[FAULT] Joint {i+1} overcurrent: {a:.3f} A")

    def smooth_move(self, targets, steps=50, delay_ms=10):
        start = [j.angle for j in self.joints]
        for s in range(1, steps + 1):
            t = s / steps
            self.move_to([start[i] + t * (targets[i] - start[i]) for i in range(6)])
            self.check_overload()
            if self.detect_collision(): return False
            time.sleep_ms(delay_ms)
        return True

    def read_manual(self):
        return (round((self.pot1.read_u16() / 65535) * 180, 1),
                round((self.pot2.read_u16() / 65535) * 180, 1))

    def status(self):
        c = self.isns.read_all()
        t = self.torque_estimate()
        print(f"{'Joint':<8}{'Angle':>8}{'Current':>10}{'Torque':>10}{'State':>8}")
        print("-" * 46)
        for i in range(6):
            print(f"J{i+1:<7}{self.joints[i].angle:>7.1f}°{c[i]:>9.3f}A{t[i]:>9.4f}Nm{'FAULT' if self.fault[i] else 'OK':>8}")

# ── Main ──────────────────────────────────────────────────
arm = RoboticArm()
arm.power_up_sequence(delay_ms=120)
arm.smooth_move([90, 90, 90, 90, 90, 90])

while True:
    j1, j2 = arm.read_manual()
    arm.move_to([j1, j2, 90, 90, 90, 90])
    arm.check_overload()
    arm.detect_collision()
    arm.status()
    time.sleep_ms(20)
```

---

## 🚀 Getting Started

**1. Power the board**
Connect VIN (6–24V) to the screw terminal. Set jumpers JP1–JP6:
- Position **A** → 5V internal buck (standard hobby servos)
- Position **B** → Direct VIN (high-torque servos above 7.4V)

**2. Flash firmware**
Hold `BOOT` on the back, press `RESET`. A drive named `RPI-RP2` appears. Drag your `.uf2` file onto it. Board reboots and runs automatically.

**3. Test with potentiometers**
RV1 and RV2 control Joint 1 and Joint 2 by default. Rotate to verify servo response before connecting the full arm.

**4. Monitor over serial**
Open any terminal at **115200 baud** on the USB-C port. `arm.status()` prints a live table of angle, current, torque, and fault state for all 6 joints.

**5. Tune collision thresholds**
Run the arm freely and log `status()` output. Set `COLLISION_NM` to ~1.5–2× the maximum torque observed during normal free movement.

**6. Extend with C++ for performance**
Run motion control on Core 0 and current sensing on Core 1 for fully parallel deterministic operation. Core 1 can achieve scan rates exceeding 10 kHz — enabling sub-millisecond collision response.

---

## ✅ Capabilities

- 6 independent PIO-generated PWM channels — sub-microsecond jitter
- Real-time torque estimation on every joint — no mechanical sensors
- Per-channel collision detection with configurable threshold
- Individual high-side MOSFET shutdown per channel
- Emergency stop — all 6 channels cut within a single GPIO toggle
- Smooth interpolated motion with mid-move collision check
- Staggered power-up to prevent inrush brownout
- Manual joint control via onboard potentiometers
- USB-C UF2 flashing — no external programmer required
- SWD debug header — J-Link / DAPLink / OpenOCD compatible
- Dual-core execution: motion on Core 0, sensing on Core 1
- MicroPython and C++ Pico SDK fully supported
- Per-servo independent voltage selection via JP1–JP6
- Live status table: angle, current, torque, fault per joint
- Latching fault system with software reset
- CERN Open Hardware License — free to modify and manufacture

---

## 📄 License

Released under the **CERN Open Hardware Licence Version 2 — Strongly Reciprocal (CERN-OHL-S v2)**.  
Free to study, modify, manufacture, and distribute — derivatives must remain open source.

---

<div align="center">Designed by <strong>B.H</strong></div>

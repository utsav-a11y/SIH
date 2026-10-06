# ⚓ SAVITR
### Software-Defined Adaptive Sonar Transmitter Payload for Autonomous Underwater Vehicles

> **Smart India Hackathon 2026 · Problem Statement SIH26058 · Ministry of Earth Sciences**

**A low-power, real-time adaptive sonar transmitter payload that dynamically selects sonar waveform parameters according to changing underwater environmental conditions.**

---

## 📑 Contents

- [Smart India Hackathon](#-smart-india-hackathon)
- [Overview](#-overview)
- [Problem](#-problem)
- [Proposed Solution](#-proposed-solution)
- [System Architecture](#-system-architecture)
- [Environmental State Space](#-environmental-state-space)
- [Offline Physics Engine](#-offline-physics-engine)
- [Adaptive Lookup Table](#-adaptive-lookup-table)
- [Waveform Selection](#-waveform-selection)
- [Runtime Operation](#-runtime-operation)
- [Hardware & Software](#-hardware--software)
- [Validation](#-validation)
- [Challenges](#-challenges)
- [Impact](#-impact)
- [Innovation](#-innovation)
- [Status](#-status)
- [Roadmap](#-roadmap)
- [References](#-references)

---

## 🚨 Smart India Hackathon

| Parameter | Details |
|---|---|
| **Problem Statement ID** | SIH26058 |
| **Problem Statement** | Development of a Low-Power, Real-Time Adaptive Software-Defined Sonar Transmitter Payload for Autonomous Underwater Vehicles (AUVs) |
| **Organization** | Ministry of Earth Sciences |
| **Theme** | Hardware / Software-defined sensing |
| **Project** | SAVITR |
| **Core Concept** | Adaptive sonar waveform generation |

---

## 🌊 Overview

Autonomous Underwater Vehicles (AUVs) rely heavily on sonar systems for underwater exploration, marine mapping, object detection and situational awareness.

However, underwater acoustic conditions are highly dynamic.

Parameters such as:

- 🌊 Depth
- 🌫️ Turbidity
- 🌡️ Temperature
- 🧂 Salinity

directly influence acoustic propagation, attenuation, scattering and detection performance.

A fixed sonar configuration therefore cannot provide optimal performance across every underwater environment.

### SAVITR addresses this limitation.

SAVITR is a **software-defined adaptive sonar transmitter payload** that observes the underwater environment and dynamically determines appropriate transmitter parameters.

The system maps environmental conditions to:

- Waveform type
- Centre frequency
- Bandwidth
- Transmit amplitude
- Pulse duration
- Windowing function

The complete decision process is designed to operate with extremely low runtime overhead, making it suitable for embedded AUV platforms.

---

# 🎯 Problem

Traditional sonar transmitters commonly operate using predefined waveform configurations.

This creates several challenges:

- Fixed waveform configurations may not be optimal across changing environments
- Underwater propagation losses vary with frequency
- Turbidity can increase acoustic scattering
- Environmental conditions affect required transmit power
- Higher power consumption reduces AUV endurance
- Dynamic optimization is difficult on resource-constrained embedded hardware
- Real-time physics calculations can introduce computational overhead
- AUV payloads require compact and deterministic processing

Therefore, the key challenge is:

> **How can an AUV dynamically select an appropriate sonar transmission strategy while maintaining low computational complexity, low power consumption and real-time response?**

SAVITR approaches this problem through an **offline physics-driven adaptive lookup architecture**.

---

# 💡 Proposed Solution

SAVITR separates computationally expensive physics modelling from real-time embedded operation.

The system follows the pipeline:

```text
ENVIRONMENT
     ↓
DEPTH / TURBIDITY / TEMPERATURE / SALINITY
     ↓
BINNING & QUANTISATION
     ↓
10,000-STATE ENVIRONMENTAL LUT
     ↓
PHYSICS-BASED PARAMETER SELECTION
     ↓
WAVEFORM + FREQUENCY + BANDWIDTH
     ↓
AMPLITUDE + PULSE DURATION
     ↓
REAL-TIME TRANSMISSION
```

Instead of repeatedly executing complex acoustic models on the embedded controller, the required decisions are computed offline and stored in a compact lookup table.

The runtime controller therefore performs primarily:

1. Sensor acquisition
2. Quantisation
3. LUT addressing
4. Parameter retrieval
5. Waveform generation
6. DAC transmission

---

# 🧩 System Architecture

```text
┌──────────────────────────────────────────────┐
│          UNDERWATER ENVIRONMENT              │
│                                              │
│ Depth · Turbidity · Temperature · Salinity  │
└──────────────────────┬───────────────────────┘
                       ↓
┌──────────────────────────────────────────────┐
│          SENSOR / ADC INTERFACE              │
└──────────────────────┬───────────────────────┘
                       ↓
┌──────────────────────────────────────────────┐
│       ENVIRONMENTAL BIN QUANTISATION         │
│              10 × 10 × 10 × 10              │
└──────────────────────┬───────────────────────┘
                       ↓
┌──────────────────────────────────────────────┐
│         10,000-STATE LOOKUP TABLE            │
│                                              │
│ Waveform · Frequency · Bandwidth             │
│ Amplitude · Pulse Duration                   │
└──────────────────────┬───────────────────────┘
                       ↓
┌──────────────────────────────────────────────┐
│       SOFTWARE-DEFINED WAVEFORM ENGINE       │
└──────────────────────┬───────────────────────┘
                       ↓
┌──────────────────────────────────────────────┐
│              DAC / TRANSMITTER               │
└──────────────────────┬───────────────────────┘
                       ↓
                 SONAR PULSE
```

---

# 🌐 Environmental State Space

SAVITR models the underwater environment using four variables:

\[
E=(D,Turb,T,S)
\]

where:

| Variable | Range | Meaning |
|---|---:|---|
| **D** | 0–100 m | Depth |
| **Turb** | 0–100 NTU | Turbidity |
| **T** | 0–30 °C | Temperature |
| **S** | 0–40 ppt | Salinity |

Each variable is divided into **10 uniform bins**.

Therefore:

\[
N_s = 10^4 = 10,000
\]

possible environmental states are represented.

---

# 🧮 Offline Physics Engine

The physics engine evaluates the environmental state space offline.

The model considers underwater acoustic propagation and determines the transmitter configuration required for reliable detection.

The system incorporates:

### Acoustic propagation

- Frequency-dependent attenuation
- Environmental scattering
- Transmission loss
- Required transmit voltage
- Detection threshold

### Acoustic models

SAVITR incorporates:

- **Francois-Garrison absorption model**
- Turbidity-dependent scattering model
- Side-scan sonar equation

The demonstration model uses:

```text
TS = 10 dB
NL = 40 dB
Detection threshold = SNR ≥ 10 dB
```

The resulting calculations are used to populate the adaptive lookup table.

---

# 🗂️ Adaptive Lookup Table

The environmental state is converted into a single LUT address.

The indexing equation is:

\[
idx = 1000d + 100t + 10\theta + \sigma
\]

where:

- `d` = depth bin
- `t` = turbidity bin
- `θ` = temperature bin
- `σ` = salinity bin

Each LUT entry stores the required transmitter configuration.

### LUT output

```text
┌──────────────────────────────┐
│ Waveform Type                │
│ Centre Frequency             │
│ Bandwidth                    │
│ DAC Amplitude                │
│ Pulse Duration               │
└──────────────────────────────┘
```

The LUT contains:

**10,000 × 5 = 50,000 bytes ≈ 49 KB**

This allows the embedded system to perform rapid parameter selection without running the complete physics model at runtime.

---

# 📡 Waveform Selection

SAVITR dynamically selects the waveform according to environmental and propagation conditions.

### LFM Chirp

Selected when:

```text
Γ < 0.55
AND
Turbidity < 60 NTU
```

### Barker-13 Phase Code

Selected when:

```text
Γ ≥ 0.55
AND
Turbidity ≥ 60 NTU
```

### Geometric Sweep

Selected when:

```text
Γ < 0.45
AND
(R ≥ 60 m OR Temperature ≤ 10 °C)
```

This enables the transmitter to adapt its signal strategy rather than relying on a single fixed waveform.

---

# 📶 Frequency & Bandwidth Adaptation

The prototype evaluates a frequency set of:

```text
500 kHz
400 kHz
300 kHz
200 kHz
100 kHz
```

For the ESP32 demonstration platform, these are scaled to an appropriate demonstration range.

Bandwidth is selected using available voltage headroom:

\[
H = 20\log_{10}\left(\frac{V_{max}}{V_{req}}\right)
\]

with:

```text
Vmax = 5 V
```

### Bandwidth policy

| Headroom | Bandwidth |
|---|---|
| H > 10 dB | Wide — 100 kHz |
| 3 dB < H ≤ 10 dB | Medium — 50 kHz |
| H ≤ 3 dB | Narrow — 20 kHz |

---

# 🔊 Adaptive Amplitude & Pulse Duration

The DAC amplitude is determined from the required voltage:

\[
A_{DAC}=clamp\left(round\left(255\frac{V_{req}}{V_{max}}\right),16,255\right)
\]

Pulse duration is calculated using:

\[
\tau_{ms}=2.0+6.0\left(\frac{V_{req}}{V_{max}}\right)
\]

with limits:

```text
Minimum pulse duration = 2 ms
Maximum pulse duration = 8 ms
```

A **Hamming window** is used for waveform shaping.

This allows the transmitter to adapt both signal strength and pulse duration to the environmental state.

---

# ⚡ Runtime Operation

The runtime architecture is designed for deterministic embedded execution.

```text
ADC INPUT
   ↓
Read D / Turb / T / S
   ↓
Quantise into bins
   ↓
Calculate LUT index
   ↓
Fetch LUT configuration
   ↓
Select waveform
   ↓
Configure hardware timer
   ↓
DMA streams waveform
   ↓
DAC OUTPUT
```

The target adaptation latency is:

> **< 60 µs**

The runtime system therefore avoids repeatedly executing the complete physics engine.

---

# 💻 Hardware & Software

### Hardware

- ESP32-class embedded controller
- ADC inputs
- DAC interface
- Hardware timer
- DMA-based waveform streaming
- Sonar transmitter interface

### Software

- Embedded C/C++
- Offline physics engine
- Lookup-table generation
- Waveform generation
- Real-time parameter selection

### Core technologies

```text
Physics Modelling
        +
Lookup Tables
        +
Embedded Systems
        +
Software-Defined Waveforms
        +
Real-Time Signal Generation
```

---

# 🧠 Memory Footprint

The compact LUT architecture is designed for embedded deployment.

### LUT

```text
10,000 states × 5 parameters
≈ 50 KB
```

### Waveform arrays

```text
≈ 150 KB
```

### Total prototype footprint

```text
≈ 200 KB
```

This makes the approach suitable for resource-constrained embedded platforms.

---

# 🧪 Validation

The system is validated across representative environmental states covering variations in:

- Depth
- Turbidity
- Temperature
- Salinity
- Required transmit voltage
- Propagation conditions

Each state produces an associated transmitter configuration through the physics engine and LUT.

### Validation objectives

- Correct environmental quantisation
- Correct LUT addressing
- Consistent waveform selection
- Valid frequency selection
- Correct bandwidth adaptation
- Correct amplitude calculation
- Correct pulse-duration mapping
- Real-time execution feasibility

---

# ⚠️ Challenges

SAVITR addresses several engineering challenges:

- Highly variable underwater acoustic conditions
- Frequency-dependent absorption
- Turbidity-driven scattering
- Limited embedded memory
- Limited processing capability
- Real-time adaptation requirements
- Power constraints in AUV platforms
- Need for deterministic transmitter behaviour

The offline-LUT architecture reduces the computational burden during deployment.

---

# 🌍 Impact

SAVITR is designed to improve the adaptability of sonar transmitters used in autonomous underwater systems.

Potential applications include:

- 🌊 Autonomous underwater exploration
- 🗺️ Marine mapping
- 🔎 Underwater object detection
- 🤖 AUV sensing
- 🌐 Oceanographic missions
- 🛡️ Maritime situational awareness
- 📡 Adaptive acoustic sensing

By selecting transmission parameters according to environmental conditions, the system aims to balance:

**Detection capability ↔ Power consumption ↔ Computational cost**

---

# 🚀 Innovation

The key innovation of SAVITR is the combination of:

### 1. Physics-driven adaptation

Environmental conditions directly influence transmitter configuration.

### 2. Offline computation

Computationally expensive acoustic modelling is performed before deployment.

### 3. Compact LUT representation

10,000 environmental states are compressed into a deployable embedded lookup structure.

### 4. Software-defined sonar

Waveform parameters are selected dynamically rather than being permanently fixed.

### 5. Real-time execution

The embedded controller performs lightweight state-to-configuration mapping.

### 6. Adaptive waveform strategy

Different waveform families are selected according to propagation conditions.

---

# 📊 Prototype Status

| Component | Status |
|---|---|
| Environmental state modelling | ✅ Implemented |
| Physics-based parameter calculation | ✅ Implemented |
| Environmental quantisation | ✅ Implemented |
| 10,000-state LUT design | ✅ Implemented |
| Waveform selection logic | ✅ Implemented |
| Frequency selection | ✅ Implemented |
| Bandwidth selection | ✅ Implemented |
| Amplitude mapping | ✅ Implemented |
| Pulse-duration mapping | ✅ Implemented |
| Embedded runtime architecture | 🔄 Prototype |
| Hardware validation | 🔄 Prototype |

---

# 🛣️ Roadmap

```text
Current Prototype
       ↓
ESP32 Hardware Integration
       ↓
DAC / Transmitter Integration
       ↓
Real-Time Sensor Acquisition
       ↓
Underwater Tank Testing
       ↓
Controlled Acoustic Testing
       ↓
AUV Integration
       ↓
Open-Water Validation
```

Future development can include:

- Larger environmental datasets
- More accurate propagation models
- Additional waveform families
- Hardware-in-the-loop testing
- Adaptive learning from field observations
- Full AUV integration
- Real-world underwater validation

---

# 📚 References

The project is based on established concepts in:

- Underwater acoustic propagation
- Sonar equation modelling
- Frequency-dependent absorption
- Acoustic scattering
- Software-defined signal generation
- Embedded real-time systems
- Autonomous underwater vehicle sensing

Detailed research papers, standards and technical references are documented in the project presentation and technical documentation.

---

# 🏁 Conclusion

**SAVITR transforms environmental observations into real-time sonar transmission decisions.**

Instead of using a fixed sonar configuration, the system evaluates the underwater state and selects an appropriate:

**Waveform + Frequency + Bandwidth + Amplitude + Pulse Duration**

through a compact physics-driven lookup architecture.

The result is a **low-power, real-time, software-defined adaptive sonar transmitter architecture designed for AUV platforms.**

---

### ⚓ SAVITR
**Adaptive Intelligence for the Underwater Domain**

**Smart India Hackathon 2026 · SIH26058**

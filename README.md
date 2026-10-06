<div align="center">

<img src="assets/banner.svg" alt="SAVITR - Adaptive Sonar Transmitter Payload" width="100%"/>

<br/>

**Adaptive sonar waveform generation using an offline physics engine, environmental-state lookup table, and real-time embedded waveform synthesis.**

<br/>

![ESP32](https://img.shields.io/badge/ESP32-Embedded-blue?style=for-the-badge&logo=espressif)
![C++](https://img.shields.io/badge/C%2B%2B-Firmware-red?style=for-the-badge&logo=cplusplus)
![Python](https://img.shields.io/badge/Python-Physics%20Engine-yellow?style=for-the-badge&logo=python)
![Sonar](https://img.shields.io/badge/SONAR-Adaptive%20Transmission-0A7EA4?style=for-the-badge)
![Real Time](https://img.shields.io/badge/Real--Time-Embedded-success?style=for-the-badge)

<br/>

![SIH](https://img.shields.io/badge/Smart%20India%20Hackathon-SIH26058-orange?style=flat-square)
![Ministry](https://img.shields.io/badge/Ministry-Earth%20Sciences-blue?style=flat-square)
![Category](https://img.shields.io/badge/Category-Software%20%2B%20Embedded-blueviolet?style=flat-square)
![Prototype](https://img.shields.io/badge/Prototype-Working%20Concept-success?style=flat-square)

</div>

---

## 📑 Contents

- [Smart India Hackathon](#-smart-india-hackathon)
- [Overview](#-overview)
- [Problem](#-problem)
- [Proposed Solution](#-proposed-solution)
- [Core Innovation](#-core-innovation)
- [Environmental State Space](#-environmental-state-space)
- [10,000-State Lookup Table](#-10000-state-lookup-table)
- [Physics Engine](#-offline-physics-engine)
- [Adaptive Waveform Selection](#-adaptive-waveform-selection)
- [Frequency & Bandwidth](#-frequency--bandwidth)
- [System Architecture](#-system-architecture)
- [Embedded Runtime](#-embedded-runtime)
- [Memory & Performance](#-memory--performance)
- [Technology Stack](#-technology-stack)
- [Applications](#-applications)
- [Innovation](#-innovation)
- [Future Scope](#-future-scope)
- [Project Status](#-project-status)
- [References](#-references)

---

## 🚨 Smart India Hackathon

**Problem Statement:** Development of a Low-Power, Real-Time Adaptive Software-Defined Sonar Transmitter Payload for Autonomous Underwater Vehicles (AUVs)

| Parameter | Details |
|---|---|
| **SIH Problem Statement** | SIH26058 |
| **Ministry** | Ministry of Earth Sciences |
| **Domain** | Autonomous Underwater Vehicles |
| **Category** | Software + Embedded |
| **System** | Adaptive Software-Defined Sonar |
| **Target Platform** | ESP32 Prototype |

---

## 🌊 Overview

**SAVITR — Software-Defined Adaptive Sonar Transmitter Payload** is a low-power, real-time adaptive sonar transmission architecture designed for **Autonomous Underwater Vehicles (AUVs)**.

Instead of transmitting a fixed waveform regardless of underwater conditions, SAVITR converts environmental observations into an optimized sonar waveform configuration.

The system considers:

- 🌊 Depth
- 🌫️ Turbidity
- 🌡️ Temperature
- 🧂 Salinity
- 🔊 Acoustic attenuation
- ⚡ Required transmission voltage
- 🎯 Detection conditions

The computationally intensive physics calculations are performed **offline**, while the embedded system performs lightweight lookup and waveform-generation operations during runtime.

---

## 🎯 Problem

Underwater acoustic propagation changes with environmental conditions.

A conventional fixed-parameter transmitter may continue using the same:

```text
Frequency
   ↓
Bandwidth
   ↓
Amplitude
   ↓
Pulse Duration
   ↓
Waveform
```

even when the underwater environment changes.

This can result in inefficient transmission and unnecessary computational or power requirements.

### SAVITR Approach

```text
Environmental Conditions
          ↓
State Quantisation
          ↓
Physics-Based Lookup
          ↓
Adaptive Waveform
          ↓
Real-Time Transmission
```

---

## 💡 Proposed Solution

SAVITR uses a **physics-informed, lookup-table-driven adaptive architecture**.

```text
Environmental Sensors
        ↓
Environmental State Quantisation
        ↓
10,000-State Physics Lookup Table
        ↓
Adaptive Waveform Selection
        ↓
Frequency + Bandwidth + Amplitude
        ↓
Pulse Duration + Windowing
        ↓
Real-Time Waveform Synthesis
        ↓
DAC / Sonar Transmitter
```

The key design principle is:

> **Compute complex physics offline. Perform fast adaptation at runtime.**

---

## 🧠 Core Innovation

SAVITR separates the system into two computational layers.

### Offline Layer

```text
Physics Models
      +
Environmental State Space
      +
Sonar Equation
      +
Waveform Selection Rules
      ↓
10,000-State Lookup Table
```

### Runtime Layer

```text
Sensor Inputs
      ↓
Quantisation
      ↓
LUT Address
      ↓
Waveform Configuration
      ↓
Embedded Synthesis
      ↓
Sonar Transmission
```

This avoids repeatedly executing computationally expensive acoustic models on the microcontroller.

---

## 🌐 Environmental State Space

SAVITR models the environment using four variables.

| Variable | Range | Bins |
|---|---:|---:|
| Depth (D) | 0–100 m | 10 |
| Turbidity (Turb) | 0–100 NTU | 10 |
| Temperature (T) | 0–30 °C | 10 |
| Salinity (S) | 0–40 ppt | 10 |

Each variable is divided into **10 uniform bins**.

Therefore:

```text
10 × 10 × 10 × 10 = 10,000 states
```

---

## 🔢 LUT Addressing

Each environmental state is converted into a single lookup-table address:

```text
idx = 1000d + 100t + 10θ + σ
```

where:

```text
d  = depth bin
t  = turbidity bin
θ  = temperature bin
σ  = salinity bin
```

The system uses **centroid quantisation** to map sensor measurements to environmental bins.

---

## 🗃️ 10,000-State Lookup Table

Each environmental state maps to a precomputed waveform configuration.

Conceptually:

| Parameter | Description |
|---|---|
| `wtype` | Waveform type |
| `fidx` | Frequency index |
| `Bidx` | Bandwidth index |
| `Adac` | DAC amplitude |
| `qτ` | Quantised pulse duration |

The runtime controller retrieves the corresponding row instead of recalculating the complete physics model.

---

## 🔬 Offline Physics Engine

The offline physics engine generates the LUT before deployment.

It incorporates:

- Underwater acoustic propagation
- Frequency-dependent attenuation
- Environmental conditions
- Required transmission voltage
- Sonar detection constraints
- Waveform selection logic

### Acoustic Absorption

SAVITR uses the **Francois-Garrison absorption model**.

Additional scattering attenuation is modeled as:

```text
αscat = Ks × Turb × (f / 100)²
```

with:

```text
Ks = 5 × 10⁻⁴
```

---

## 📡 Sonar Detection Model

The demonstration model uses:

```text
TS = 10 dB
NL = 40 dB
Required SNR ≥ 10 dB
```

The system evaluates transmission requirements and uses them to determine the appropriate waveform configuration.

---

## 📶 Adaptive Waveform Selection

SAVITR supports three waveform strategies.

### 1. LFM Chirp

```text
Γ < 0.55
AND
Turbidity < 60 NTU
```

Used for relatively favorable environmental conditions.

### 2. Barker-13 Phase Code

```text
Γ ≥ 0.55
AND
Turbidity ≥ 60 NTU
```

Used for more challenging environmental conditions.

### 3. Geometric Sweep

```text
Γ < 0.45
AND
(R ≥ 60 m OR T ≤ 10 °C)
```

Provides an additional adaptive transmission strategy.

---

## 🎚️ Frequency & Bandwidth

### Candidate Frequencies

```text
500 kHz
400 kHz
300 kHz
200 kHz
100 kHz
```

### Target Frequencies

**Target Frequencies:** 100, 200, 300, 400, 500 kHz.

**PoC Validation:** The exact same adaptive logic was validated on the ESP32 at a scaled **10–20 kHz range (100 kSPS)** due to PoC hardware bandwidth limits. The digital architecture (LUT + DMA) remains identical; only the sample rate and analog front-end change for the final target.

### Bandwidth Selection

Voltage headroom:

```text
H = 20 log₁₀(Vmax / Vreq)
```

with:

```text
Vmax = 5 V
```

| Headroom | Bandwidth |
|---|---:|
| H > 10 dB | 100 kHz |
| 3 dB < H ≤ 10 dB | 50 kHz |
| H ≤ 3 dB | 20 kHz |

---

## ⚡ Amplitude & Pulse Duration

DAC amplitude:

```text
Adac = clamp(
    round(255 × Vreq / Vmax),
    16,
    255
)
```

Pulse duration:

```text
τms = 2.0 + 6.0(Vreq / Vmax)
```

with:

```text
2 ms ≤ τ ≤ 8 ms
```

### Windowing

SAVITR uses a **Hann/Blackman window applied offline for waveform shaping to suppress spectral leakage.**

---

## 🏗️ System Architecture

```text
┌─────────────────────────────┐
│    Environmental Sensors    │
│ Depth / Turbidity / Temp / S│
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│      ADC Acquisition        │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Environmental Quantisation  │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│     10,000-State LUT        │
│   Physics-Based Mapping     │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│   Adaptive Waveform Logic   │
│ LFM / Barker-13 / Sweep     │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Frequency / Bandwidth / Amp │
│       Pulse Duration        │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│   Real-Time Waveform Engine │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│       DAC + DMA + Timer     │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│      SONAR TRANSMITTER      │
└─────────────────────────────┘
```

---

## ⚙️ Embedded Runtime

The ESP32 performs a lightweight runtime pipeline:

```text
ADC Read
   ↓
Bin Quantisation
   ↓
LUT Index Calculation
   ↓
LUT Row Fetch
   ↓
Waveform Parameters
   ↓
Hardware Timer
   ↓
DMA Waveform Streaming
   ↓
DAC Output
```

The embedded controller therefore avoids repeatedly running the complete physics engine.

---

## 🚀 Runtime Configuration

The LUT returns:

```text
## 🗃️ 10,000-State Lookup Table

Each environmental state maps to a precomputed 4-byte waveform configuration:

`[wtype, fidx, Adac, qτ]`

**Memory Optimization:** `fidx` inherently couples Center Frequency and Bandwidth. This allows 4 bytes to command all 5 adaptive parameters, keeping the LUT strictly at **40 KB** to fit within ESP32 Flash limits.
```

The hardware timer controls pulse duration while DMA streams the generated waveform to the DAC.

### Target Adaptation Latency

```text
< 60 µs
```

---

## 💾 Memory & Performance

| Component | Approximate Size |
|---|---:|
| LUT | 10,000 × 4 bytes |
| LUT memory | ~40 KB |
| LUT size | ~39 KB |
| Waveform arrays | ~150 KB |
| Total | ~190 KB |
| Target adaptation latency | < 60 µs |

The compact LUT representation enables complex environmental decisions to be stored in a form suitable for embedded execution.

---

## 🔄 End-to-End Operation

```text
UNDERWATER ENVIRONMENT
          ↓
Environmental Sensors
          ↓
ADC Acquisition
          ↓
State Quantisation
          ↓
LUT Address
          ↓
10,000-State Physics LUT
          ↓
Waveform Selection
      ↙      ↓      ↘
    LFM    Barker   Sweep
      ↘      ↓      ↙
 Frequency + Bandwidth
          ↓
 Amplitude + Duration
          ↓
 Hamming Window
          ↓
 Waveform Synthesis
          ↓
      DAC / DMA
          ↓
 SONAR TRANSMISSION
```

---

## 🧪 Example Environmental States

| Depth | Turbidity | Temperature | Salinity | Example Waveform |
|---:|---:|---:|---:|---|
| 10 m | 20 NTU | 25 °C | 35 ppt | LFM |
| 40 m | 50 NTU | 20 °C | 35 ppt | LFM |
| 70 m | 80 NTU | 15 °C | 37 ppt | Barker-13 |
| 90 m | 30 NTU | 8 °C | 38 ppt | Geometric Sweep |

---

## 🛠️ Technology Stack

### Embedded

- ESP32
- C++
- ADC
- DAC
- Hardware Timer
- DMA

### Physics & Simulation

- Python
- Sonar equation
- Francois-Garrison absorption model
- Scattering model
- Lookup-table generation

### Signal Processing

- LFM Chirp
- Barker-13 Phase Coding
- Geometric Sweep
- Hamming Window
- Adaptive bandwidth selection

---

## 🌊 Applications

SAVITR is designed for potential use in:

- Autonomous Underwater Vehicles
- Marine Mapping
- Seafloor Survey
- Oceanographic Research
- Underwater Monitoring
- Autonomous Navigation
- Subsea Exploration
- Distributed Underwater Sensor Networks

---

## 💡 Innovation

SAVITR combines:

```text
Physics-Based Modeling
        +
Environmental State Quantisation
        +
10,000-State Lookup Table
        +
Software-Defined Waveforms
        +
Real-Time Embedded Synthesis
```

### Key Idea

> **Compute complex physics once. Adapt the waveform continuously.**

Compared with a fixed-parameter transmitter, SAVITR introduces environment-aware selection of waveform, frequency, bandwidth, amplitude and pulse duration.

---

## 📊 Conventional vs SAVITR

| Conventional Transmitter | SAVITR |
|---|---|
| Fixed waveform | Adaptive waveform |
| Fixed frequency | Environment-dependent frequency |
| Fixed bandwidth | Adaptive bandwidth |
| Fixed amplitude | Adaptive amplitude |
| Heavy runtime computation | Offline physics + LUT |
| Static configuration | Software-defined configuration |
| Limited environmental adaptation | 10,000-state environmental mapping |

---

## 🔮 Future Scope

Future development can extend SAVITR toward:

### Hardware

- Real underwater transducer
- Power amplifier
- High-speed DAC
- Waterproof sensor package
- Full AUV integration

### Software

- Larger environmental state spaces
- Dynamic LUT updates
- Machine-learning-assisted optimization
- Online environmental estimation
- Mission-level adaptive control

### Closed-Loop Sonar

```text
Environmental Sensors
        ↓
Adaptive Transmitter
        ↓
Underwater Channel
        ↓
Echo / Received Signal
        ↓
Signal Processing
        ↓
Detection Performance
        ↓
Feedback
        ↺
Adaptive Waveform
```

This can evolve SAVITR toward a closed-loop intelligent sonar transmission system.

### Runtime Scalar Adaptation: Target Range & Velocity

While the 4D LUT handles the *water conditions* (Depth, Turbidity, Temp, Salinity), SAVITR handles dynamic platform constraints like **Target Range** and **AUV Velocity** as post-fetch runtime scalars.

**Why not add Range to the LUT?**
Adding Target Range (10 bins) and Velocity (10 bins) to the 4D state space would create $10^6$ (1,000,000) states. At 4 bytes per state, this requires **4 MB of Flash memory**, which exceeds the ESP32's capacity.

**The Solution: Post-Fetch Scalars**
Instead of bloating the LUT, the firmware applies Range and Velocity as mathematical scalars *after* fetching the 4-byte configuration:
*   **Range Scaling:** Dynamically scales the transmit amplitude (`Adac`) and pulse duration (`qτ`) based on the target distance to maintain the Sonar Equation margin.
*   **Velocity Compensation:** Applies a Doppler correction factor for high-speed AUV maneuvers.

This allows SAVITR to support full **6-parameter adaptation** (Environment + Range + Velocity) while keeping the core LUT strictly at a lightweight **40 KB**.

### 4th Waveform Family: Hyperbolic FM (HFM)

For high-velocity AUV maneuvers, SAVITR plans to integrate **Hyperbolic Frequency Modulation (HFM)** as a 4th waveform family. Unlike LFM, HFM is inherently Doppler-tolerant, maintaining pulse compression performance under high relative velocities. Like the other waveforms, HFM will be pre-computed by the offline Python physics engine and streamed via zero-CPU DMA.

---

## 📁 Project Structure

```text
SAVITR/
│
├── assets/
│   └── banner.svg
│
├── firmware/
│   ├── main.cpp
│   ├── waveform_generator.cpp
│   ├── waveform_generator.h
│   ├── lut.cpp
│   └── lut.h
│
├── physics_engine/
│   ├── sonar_model.py
│   ├── absorption.py
│   ├── scattering.py
│   ├── waveform_selection.py
│   └── generate_lut.py
│
├── data/
│   └── lookup_table/
│
├── simulation/
│
├── docs/
│
└── README.md
```

---

## 🚧 Challenges

The prototype addresses several constraints associated with AUV payloads:

- Limited onboard processing
- Limited memory
- Power constraints
- Changing underwater conditions
- Frequency-dependent attenuation
- Real-time adaptation requirements
- Hardware voltage limitations
- Transducer integration

---

## 📈 Expected Impact

SAVITR aims to provide:

- ⚡ Low computational overhead
- 🔄 Fast waveform reconfiguration
- 🌊 Environmental adaptability
- 🔋 More efficient transmission decisions
- 💻 Compact embedded implementation
- 📡 Software-defined sonar flexibility
- 🤖 AUV-ready adaptive architecture

---

## 🟢 Project Status

| Module | Status |
|---|---|
| Environmental state model | 🟢 |
| Physics engine concept | 🟢 |
| 10,000-state LUT | 🟢 |
| Waveform selection logic | 🟢 |
| Adaptive parameter selection | 🟢 |
| ESP32 runtime architecture | 🟢 |
| Real-time waveform synthesis | 🟡 |
| Physical transducer integration | 🟡 |
| Full AUV deployment | 🔵 |

**Legend:**  
🟢 Implemented / demonstrated concept  
🟡 Prototype / integration stage  
🔵 Future development

---

## 📚 References

The SAVITR architecture is based on established concepts in:

- Underwater acoustic propagation
- Sonar equation and detection theory
- Acoustic absorption modeling
- Software-defined signal generation
- Embedded waveform synthesis
- Autonomous underwater vehicle systems
- Adaptive sonar transmission

Detailed research-paper references can be maintained in the project documentation.

---

## 🏆 Smart India Hackathon 2026

<div align="center">

**SAVITR**

### Software-Defined Adaptive Sonar Transmitter Payload

**SIH 2026 • SIH26058 • Ministry of Earth Sciences**

<br/>

🌊 **ADAPT • COMPUTE • TRANSMIT**

<br/>

*Making sonar transmission environment-aware.*

</div>

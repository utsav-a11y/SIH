# ☀️ SAVITR
## Low-Power, Real-Time Adaptive Software-Defined Sonar Transmitter Payload for Autonomous Underwater Vehicles

> **Adaptive sonar waveform generation using an offline physics engine, environmental-state lookup table, and real-time embedded waveform synthesis.**

<p align="center">

![ESP32](https://img.shields.io/badge/ESP32-Embedded-blue?style=for-the-badge&logo=espressif)
![C/C++](https://img.shields.io/badge/C%2FC%2B%2B-Firmware-red?style=for-the-badge&logo=cplusplus)
![Python](https://img.shields.io/badge/Python-Physics%20Engine-yellow?style=for-the-badge&logo=python)
![Sonar](https://img.shields.io/badge/SONAR-Adaptive%20Transmission-0A7EA4?style=for-the-badge)
![Real Time](https://img.shields.io/badge/Real--Time-%3C60%C2%B5s-success?style=for-the-badge)
![SIH 2026](https://img.shields.io/badge/Smart%20India%20Hackathon-2026-orange?style=for-the-badge)

</p>

<p align="center">

**Smart India Hackathon 2026 · SIH26058 · Ministry of Earth Sciences**

</p>

---

## 📑 Contents

- [Smart India Hackathon](#-smart-india-hackathon)
- [Overview](#-overview)
- [Problem](#-problem)
- [Proposed Solution](#-proposed-solution)
- [System Architecture](#-system-architecture)
- [Environmental State Space](#-environmental-state-space)
- [Offline Physics Engine](#-offline-physics-engine)
- [10,000-State Lookup Table](#-10000-state-lookup-table)
- [Adaptive Waveform Selection](#-adaptive-waveform-selection)
- [Runtime Pipeline](#-runtime-pipeline)
- [Embedded Implementation](#-embedded-implementation)
- [Technical Specifications](#-technical-specifications)
- [Validation](#-validation)
- [Advantages](#-advantages)
- [Challenges](#-challenges)
- [Future Roadmap](#-future-roadmap)
- [References](#-references)

---

# 🚨 Smart India Hackathon

| Parameter | Details |
|---|---|
| **Problem Statement ID** | SIH26058 |
| **Problem Statement** | Development of a Low-Power, Real-Time Adaptive Software-Defined Sonar Transmitter Payload for Autonomous Underwater Vehicles (AUVs) |
| **Organization** | Ministry of Earth Sciences |
| **Hackathon** | Smart India Hackathon 2026 |
| **Category** | Hardware / Software |
| **Domain** | Underwater Technology / Sonar / Embedded Systems |

---

# 🌊 Overview

Autonomous Underwater Vehicles (AUVs) operate in highly variable underwater environments where **depth, turbidity, temperature and salinity** can significantly affect acoustic propagation.

A conventional sonar transmitter generally operates with a predetermined waveform configuration.

This creates a fundamental limitation:

> **The transmitted waveform may not remain optimal as the underwater environment changes.**

SAVITR addresses this problem using a **software-defined adaptive sonar transmitter architecture**.

The system:

- Measures environmental parameters
- Converts them into discrete environmental states
- Uses an offline physics engine to characterize acoustic conditions
- Maps each environmental state to an optimized transmission configuration
- Stores the resulting configurations in a compact lookup table
- Performs real-time waveform selection on an ESP32
- Generates the required waveform using a DAC
- Adapts transmission parameters without performing expensive physics calculations during runtime

---

# 🎯 Problem

AUV sonar systems must operate under changing underwater conditions.

The acoustic environment changes with:

- 🌊 **Depth**
- 🌫️ **Turbidity**
- 🌡️ **Temperature**
- 🧂 **Salinity**

These parameters influence:

- Acoustic absorption
- Scattering
- Propagation loss
- Required transmission amplitude
- Detection performance
- Suitable waveform characteristics

A conventional fixed waveform system cannot efficiently adapt to these changing conditions.

### Key challenges

- High computational cost of real-time acoustic modelling
- Limited embedded processing resources
- Power constraints on AUV platforms
- Changing underwater propagation conditions
- Requirement for deterministic real-time behaviour
- Need for adaptive waveform selection
- Need to maintain sufficient detection performance

---

# 💡 Proposed Solution

SAVITR introduces an **offline physics-driven state-to-waveform mapping architecture**.

Instead of repeatedly running complex acoustic calculations on the embedded controller:

```text
Environmental Sensors
        │
        ▼
┌─────────────────────┐
│ Environmental State │
│ D · Turb · T · S    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Offline Physics     │
│ Engine               │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 10,000-State LUT    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ ESP32 Runtime       │
│ State Lookup        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Waveform Generator  │
│ DAC + DMA + Timer   │
└──────────┬──────────┘
           │
           ▼
      SONAR OUTPUT
```

The expensive calculations are performed **offline**, while the embedded system performs only a lightweight lookup and waveform-generation operation.

---

# 🧭 System Architecture

```text
                    UNDERWATER ENVIRONMENT
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
          Depth          Turbidity       Temperature
             │                │                │
             └────────────────┼────────────────┘
                              │
                         Salinity
                              │
                              ▼
                  ┌────────────────────┐
                  │ State Quantization │
                  │   10 × 10 × 10 × 10│
                  └─────────┬──────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ 10,000-State Physics │
                 │        LUT           │
                 └──────────┬───────────┘
                            │
                            ▼
                ┌────────────────────────┐
                │ Waveform Configuration │
                │                        │
                │ Type                   │
                │ Frequency              │
                │ Bandwidth              │
                │ Amplitude              │
                │ Pulse Duration         │
                └───────────┬────────────┘
                            │
                            ▼
                       ESP32 Runtime
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ▼                   ▼
              Hardware             DMA
               Timer              Streaming
                  │                   │
                  └─────────┬─────────┘
                            ▼
                           DAC
                            │
                            ▼
                    ADAPTIVE SONAR
                       TRANSMISSION
```

---

# 🌐 Environmental State Space

The environmental state is represented as:

```text
E = (D, Turb, T, S)
```

Where:

| Parameter | Symbol | Range |
|---|---:|---:|
| Depth | `D` | 0–100 m |
| Turbidity | `Turb` | 0–100 NTU |
| Temperature | `T` | 0–30 °C |
| Salinity | `S` | 0–40 ppt |

Each parameter is divided into **10 uniform bins**.

Therefore:

```text
10 × 10 × 10 × 10
= 10,000 environmental states
```

Each state is represented by a centroid value for runtime quantization.

---

# 🧮 LUT Addressing

Each environmental state is converted into a single lookup-table index:

```text
idx = 1000d + 100t + 10θ + σ
```

Where:

```text
d     → depth bin
t     → turbidity bin
θ     → temperature bin
σ     → salinity bin
```

This allows the ESP32 to directly access the configuration associated with the current environmental state.

### LUT Structure

Each state stores:

```text
[wtype, fidx, Bidx, Adac, qτ]
```

Where:

| Field | Description |
|---|---|
| `wtype` | Waveform type |
| `fidx` | Frequency index |
| `Bidx` | Bandwidth index |
| `Adac` | DAC amplitude |
| `qτ` | Quantized pulse duration |

---

# ⚙️ Offline Physics Engine

The physics engine evaluates the acoustic behaviour of each environmental state before deployment.

The model incorporates:

### Acoustic propagation

- Absorption
- Scattering
- Transmission loss
- Required voltage
- Detection condition

### Absorption model

The system uses the:

**Francois–Garrison absorption model**

### Scattering model

```text
αscat = Ks × Turb × (f / 100)²
```

with:

```text
Ks = 5 × 10⁻⁴
```

### Detection criterion

The sonar detection condition is:

```text
SNR ≥ 10 dB
```

with the defined demo parameters:

```text
TS = 10 dB
NL = 40 dB
```

---

# 📦 10,000-State Lookup Table

The complete environmental space is precomputed offline.

```text
10,000 states
        │
        ▼
Physics evaluation
        │
        ▼
Waveform optimisation
        │
        ▼
Configuration generation
        │
        ▼
LUT
```

### Memory footprint

```text
10,000 × 5 bytes
≈ 50,000 bytes
≈ 49 KB
```

Additional waveform arrays require approximately:

```text
≈ 150 KB
```

Total approximate memory requirement:

```text
≈ 200 KB
```

This makes the approach suitable for resource-constrained embedded deployment.

---

# 📡 Adaptive Waveform Selection

SAVITR selects the waveform according to the calculated acoustic/environmental state.

### LFM Chirp

Selected when:

```text
Γ < 0.55
AND
Turbidity < 60
```

### Barker-13 Phase Code

Selected when:

```text
Γ ≥ 0.55
AND
Turbidity ≥ 60
```

### Geometric Sweep

Selected when:

```text
Γ < 0.45
AND
(
    Range ≥ 60 m
    OR
    Temperature ≤ 10 °C
)
```

This allows the transmitter to adapt its waveform strategy rather than relying on a single fixed transmission scheme.

---

# 📶 Frequency Selection

The supported frequency set is:

```text
{500, 400, 300, 200, 100} kHz
```

For the ESP32 demonstration platform, frequencies are scaled by a factor of 10:

```text
10 – 50 kHz
```

This allows the same adaptive decision architecture to be demonstrated using hardware suitable for the prototype.

---

# 📊 Bandwidth Selection

Bandwidth is selected using the available voltage headroom.

```text
H = 20 log₁₀(Vmax / Vreq)
```

with:

```text
Vmax = 5 V
```

### Decision logic

| Headroom | Bandwidth |
|---|---|
| `H > 10 dB` | Wide → 100 kHz |
| `3 dB < H ≤ 10 dB` | Medium → 50 kHz |
| `H ≤ 3 dB` | Narrow → 20 kHz |

---

# 🔊 Amplitude & Pulse Duration

The DAC amplitude is calculated as:

```text
Adac = clamp(
    round(255 × Vreq / Vmax),
    16,
    255
)
```

Pulse duration:

```text
τms = 2.0 + 6.0 × (Vreq / Vmax)
```

with:

```text
τmin = 2 ms
τmax = 8 ms
```

A **Hamming window** is used for waveform shaping.

---

# ⚡ Runtime Pipeline

Once deployed, the ESP32 does not need to repeat the complete physics calculation.

Instead:

```text
ADC Sensors
    │
    ▼
Read D / Turb / T / S
    │
    ▼
Quantize → Environmental Bins
    │
    ▼
Calculate LUT Index
    │
    ▼
Fetch LUT Row
    │
    ├── Waveform Type
    ├── Frequency
    ├── Bandwidth
    ├── Amplitude
    └── Pulse Duration
    │
    ▼
Hardware Timer
    │
    ▼
DMA
    │
    ▼
DAC
    │
    ▼
Adaptive Sonar Pulse
```

---

# ⏱️ Real-Time Operation

The architecture is designed for deterministic embedded execution.

### Runtime adaptation latency

```text
< 60 µs
```

The runtime path avoids expensive:

- Physics calculations
- Optimisation loops
- Acoustic model evaluation
- Dynamic waveform selection algorithms

Instead, the embedded controller performs:

```text
SENSE → QUANTIZE → LOOKUP → GENERATE
```

---

# 🧠 Embedded Implementation

The ESP32 performs the real-time portion of the system.

### Runtime components

| Component | Function |
|---|---|
| ADC | Environmental sensor acquisition |
| Quantizer | Converts measurements into bins |
| LUT | Stores precomputed configurations |
| Hardware Timer | Controls pulse timing |
| DMA | Streams waveform samples |
| DAC | Generates analogue waveform |
| Firmware | Coordinates the complete runtime pipeline |

---

# 🛠️ Technology Stack

<p align="center">

![ESP32](https://img.shields.io/badge/ESP32-Embedded%20Controller-blue?style=flat-square)
![Python](https://img.shields.io/badge/Python-Physics%20Engine-yellow?style=flat-square)
![C++](https://img.shields.io/badge/C%2FC%2B%2B-Firmware-red?style=flat-square)
![DAC](https://img.shields.io/badge/DAC-Waveform%20Generation-purple?style=flat-square)
![DMA](https://img.shields.io/badge/DMA-Real--Time%20Streaming-green?style=flat-square)

</p>

### Core technologies

- **ESP32**
- **C/C++ embedded firmware**
- **Python-based offline physics engine**
- **DAC waveform generation**
- **DMA waveform streaming**
- **Hardware timers**
- **Lookup-table optimisation**

---

# 🧪 Validation

The prototype architecture can be evaluated across representative environmental states.

Validation focuses on:

- Correct environmental-state quantization
- Correct LUT addressing
- Correct waveform selection
- Frequency selection
- Bandwidth selection
- DAC amplitude calculation
- Pulse-duration calculation
- Runtime latency
- Memory footprint
- Adaptive behaviour across environmental conditions

---

# 🚀 Advantages

### ⚡ Low Runtime Computation

Complex physics calculations are moved offline.

### 🧠 Adaptive

Waveform parameters change according to the environmental state.

### 🔋 Low-Power Friendly

The embedded controller performs lightweight deterministic operations.

### ⏱️ Real-Time

The lookup-based runtime path targets sub-60 µs adaptation latency.

### 📦 Compact

The complete state-to-waveform LUT is approximately 49 KB.

### 🔧 Software-Defined

The waveform strategy can be changed through firmware/LUT updates without redesigning the entire transmitter architecture.

### 🌊 Environment-Aware

The transmitter considers:

```text
Depth
Turbidity
Temperature
Salinity
```

rather than operating with a fixed configuration.

---

# ⚠️ Challenges

The major engineering challenges include:

- Accurate acoustic modelling
- Embedded memory constraints
- Real-time waveform generation
- Environmental sensor quantization
- Maintaining deterministic latency
- Balancing detection performance and power consumption
- Translating theoretical acoustic models into an embedded implementation

---

# 💡 Innovation

The central innovation of SAVITR is the separation of:

```text
PHYSICS INTELLIGENCE
        ↓
OFFLINE
        ↓
LOOKUP TABLE
        ↓
REAL-TIME EMBEDDED EXECUTION
```

Instead of forcing a resource-constrained AUV controller to repeatedly solve computationally expensive acoustic models, SAVITR converts the physics into a compact **state-to-waveform intelligence layer**.

This creates a bridge between:

**Acoustic Physics → AI/Algorithmic Decision → Embedded Real-Time Transmission**

---

# 🗺️ Roadmap

```text
[x] Environmental state-space definition
[x] Physics-based modelling
[x] 10,000-state LUT architecture
[x] Waveform-selection logic
[x] Embedded runtime architecture
[x] ESP32 demonstration concept
[ ] Hardware-integrated underwater testing
[ ] Hydrophone-based closed-loop validation
[ ] Expanded environmental state resolution
[ ] Hardware transmitter integration
[ ] AUV field deployment
```

---

# 📚 References

The project is based on research and established models in:

- Underwater acoustic propagation
- Sonar equation modelling
- Acoustic absorption
- Scattering
- Adaptive waveform design
- Autonomous underwater vehicle systems
- Embedded real-time signal generation
- Francois–Garrison absorption modelling

---

# 👨‍💻 Project

**SAVITR — Software-Defined Adaptive Sonar Transmitter Payload**

**Smart India Hackathon 2026**

**Problem Statement:** SIH26058

**Organization:** Ministry of Earth Sciences

> *Making sonar transmission adaptive, computationally efficient, and environment-aware for next-generation autonomous underwater systems.*

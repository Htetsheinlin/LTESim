# LTESim
LTESim

It models:

* LTE logical architecture (inspired by 3rd Generation Partnership Project standards)
* Simulator internal software architecture
* Multi-cell + traffic + handover + scheduler integration

---

# LTE System-Level Simulator Architecture (Markdown + Mermaid)

---

# 1. High-Level LTE Logical Architecture

```mermaid
flowchart LR

    UE1[User Equipment 1]
    UE2[User Equipment 2]
    UE3[User Equipment N]

    eNB1[eNodeB - Cell 1]
    eNB2[eNodeB - Cell 2]
    eNB3[eNodeB - Cell N]

    MME[Mobility Management Entity]
    SGW[Serving Gateway]
    PGW[Packet Data Network Gateway]
    HSS[Home Subscriber Server]

    UE1 --> eNB1
    UE2 --> eNB1
    UE3 --> eNB2

    eNB1 --> MME
    eNB2 --> MME
    eNB3 --> MME

    MME --> HSS
    MME --> SGW
    SGW --> PGW
```

Note for Simulator:

You will NOT simulate full EPC signaling in detail.
You will abstract:

* Authentication → simple approval model
* IP allocation → dummy assignment
* Core delay → fixed latency model

Because this is **system-level simulation**, not packet-level.

---

# 2. Simulator Software Architecture (Core Design)

Now we move from telecom architecture to software architecture.

```mermaid
flowchart TD

    SimulationEngine

    subgraph Network
        CellManager
        ChannelModel
        InterferenceModel
    end

    subgraph Users
        UserManager
        MobilityModel
        TrafficModel
    end

    subgraph Radio
        Scheduler
        ResourceAllocator
        CQIMapping
    end

    subgraph Control
        HandoverManager
        AdmissionControl
    end

    subgraph Metrics
        ThroughputCalculator
        BlockingProbability
        DelayTracker
        FairnessIndex
    end

    SimulationEngine --> Network
    SimulationEngine --> Users
    SimulationEngine --> Radio
    SimulationEngine --> Control
    SimulationEngine --> Metrics
```

---

# 3. Main Simulation Time Loop

This is the heart of the simulator.

```mermaid
flowchart TD

    Start --> UpdateMobility
    UpdateMobility --> ComputePathLoss
    ComputePathLoss --> CalculateSINR
    CalculateSINR --> MapToCQI
    MapToCQI --> RunScheduler
    RunScheduler --> AllocateRB
    AllocateRB --> UpdateThroughput
    UpdateThroughput --> CheckHandover
    CheckHandover --> LogMetrics
    LogMetrics --> NextTTI
    NextTTI --> UpdateMobility
```

TTI = 1 ms (Transmission Time Interval)

---

# 4. Multi-Cell Interference Model

```mermaid
flowchart LR

    UE --> ServingCell
    UE --> NeighborCell1
    UE --> NeighborCell2

    NeighborCell1 --> InterferencePower
    NeighborCell2 --> InterferencePower

    ServingCell --> SignalPower

    SignalPower --> SINR
    InterferencePower --> SINR
```

Formula implemented in ChannelModel:

```
SINR = Signal / (Interference + Noise)
```

---

# 5. Handover Decision Flow (Event A3 Model)

```mermaid
flowchart TD

    MeasureRSRP --> CompareSignals
    CompareSignals -->|Target > Serving + Offset| StartTTT
    StartTTT -->|TTT expired| TriggerHandover
    TriggerHandover --> SwitchServingCell
```

This is simplified LTE Event A3 logic.

---

# 6. Traffic Flow Model (Erlang-Based)

```mermaid
flowchart TD

    UserArrival --> AdmissionCheck
    AdmissionCheck -->|Resources Available| AllocateRB
    AdmissionCheck -->|No Resources| BlockUser
    AllocateRB --> StartSession
    StartSession --> SessionTimer
    SessionTimer --> ReleaseResources
```

Blocking Probability calculated via:

* Erlang B formula
* Real-time simulation result comparison

---

# 7. Object-Oriented Class Relationship (For Python / MATLAB)

```mermaid
classDiagram

    class SimulationEngine {
        +run()
        +initialize()
        +update()
    }

    class Cell {
        +id
        +power
        +bandwidth
        +users[]
    }

    class User {
        +position
        +sinr
        +cqi
        +throughput
    }

    class ChannelModel {
        +computePathLoss()
        +computeSINR()
    }

    class Scheduler {
        +allocateRB()
    }

    class HandoverManager {
        +checkA3Event()
    }

    SimulationEngine --> Cell
    SimulationEngine --> User
    SimulationEngine --> ChannelModel
    SimulationEngine --> Scheduler
    SimulationEngine --> HandoverManager
```

---

# 8. Mapping to Real LTE Components

| Real LTE | Simulator Equivalent  |
| -------- | --------------------- |
| eNodeB   | Cell class            |
| UE       | User class            |
| PHY      | ChannelModel          |
| MAC      | Scheduler             |
| RRC      | HandoverManager       |
| EPC      | Abstracted core logic |

This abstraction follows 3GPP conceptual layering but keeps simulation efficient.

---

#  Final Simulator Structure (Directory Example)

```
lte_simulator/
│
├── main.py
├── config.py
│
├── network/
│   ├── cell.py
│   ├── channel_model.py
│   ├── interference.py
│
├── users/
│   ├── user.py
│   ├── mobility.py
│   ├── traffic.py
│
├── radio/
│   ├── scheduler.py
│   ├── cqi_mapping.py
│
├── control/
│   ├── handover.py
│   ├── admission.py
│
├── metrics/
│   ├── throughput.py
│   ├── blocking.py
│   ├── fairness.py
│
└── results/
    ├── plots/
    ├── logs/
```

---

#  What This Architecture Achieves

The simulator will be able to:

* Support N cells
* Support M users
* Simulate mobility
* Simulate interference
* Model traffic statistically
* Compare scheduler algorithms
* Evaluate handover performance
* Produce research-grade graphs

---


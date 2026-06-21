
# Deep-Dive Technical Analysis of BYD’s E/E, Compute, Software, and AI Architecture

## Executive Summary

BYD has evolved from a largely distributed ECU-based E/E architecture to a domain-centralized platform embodied in e‑Platform 3.0, with clear movement toward higher compute centralization and mixed‑criticality consolidation for its premium brands (Denza, Yangwang, Fangchengbao). The company’s vertically integrated strategy (FinDreams for powertrain, batteries, and domain controllers) and self‑developed BYD OS underpin a cost‑optimized SDV‑style architecture that emphasizes rapid rollout of OTA features and high ADAS penetration across the lineup.[1][2][3]

On the software side, BYD combines an in‑house vehicle OS (BYD OS) with Android‑based DiLink IVI stacks, and leverages safety‑grade RTOS and AUTOSAR‑style methods on MCUs in powertrain and chassis domains, especially where NVIDIA Orin or Chinese AI SoCs (Horizon Journey series) are used under QNX or Linux for high‑performance AD/ADAS compute. Evidence points to a pragmatic, hybrid architecture rather than a fully proprietary Tesla‑style monolith or a pure standards stack.[4][3][5][1]

In ADAS/AD, BYD relies heavily on an ecosystem of Chinese and global partners (Horizon Robotics, Momenta, Baidu, DJI, Holomatic, Sensetime) combined with in‑house BOS/BYD OS‑based driving stacks, and is now rolling out its “God’s Eye” ADAS in multiple performance tiers (DiPilot 100/300/600) powered by heterogeneous compute (Journey 5/6, NVIDIA Orin) to cover the Ocean/Dynasty mass segments and Yangwang/Fangchengbao premium segments.[6][7][8][4]

BYD’s connectivity and OTA infrastructure is mature and aggressive: e‑Platform 3.0 supports full‑vehicle OTA and is tightly coupled with the DiLink/vehicle OS stack; public data indicates hundreds of OTA pushes per year across Ocean/Dynasty, combined with regionally communicated cadences of “every few months” for consumer‑visible packages. Its organization is strongly hardware‑software vertically integrated, with BYD OS, domain controllers, power electronics, and batteries largely self‑developed, enabling both rapid feature iteration and structurally lower BOM versus peers.[3][9][10][1]

The net result is a highly “software‑enhanced, vertically integrated EV platform” that is not yet as software‑first and centralized as Tesla’s latest generation, but is arguably ahead of most legacy OEMs and competitive with Xiaomi and other Chinese SDV entrants in terms of OTA cadence, ADAS coverage, and cost‑performance.

***

## 1. E/E & Hardware Compute Architecture Evolution

### 1.1 Historical evolution of BYD E/E architecture

Early BYD vehicles followed the classic distributed ECU paradigm, with multiple CAN‑connected controllers for body, powertrain, and chassis functions, similar to legacy ICE OEMs; BYD’s vertical integration focused more on power electronics and batteries than on architectural innovation at that time. As BEVs and DM‑i/DM‑p hybrids scaled, limitations of distributed ECUs led BYD to converge toward domain‑based architectures, consolidating body, powertrain, and cockpit functions into larger controllers while retaining legacy CAN‑based sub‑ECUs for actuators and low‑level control.[2]

The introduction of e‑Platform 3.0 in 2021 marks BYD’s explicit pivot from distributed E/E to an integrated, domain‑controlled architecture with four major domain control systems: smart powertrain, vehicle control/body, smart cockpit, and intelligent driving. Official materials repeatedly position this as an upgrade from conventional distributed EEA to “domain centralized” architecture, emphasizing improved interaction efficiency and system upgradability. The platform is now the base for most current Ocean and Dynasty series vehicles, as well as exported models like the Atto 3.[11][12][13][1]

For premium brands such as Yangwang, BYD builds on e‑Platform 3.x with additional domain consolidation and higher compute density (e.g., Orin‑based intelligent driving domain, four‑motor e⁴ power domain), pointing toward a zonal‑style future where a small number of high‑performance controllers govern larger physical zones and multiple functional domains. Public information still describes the architecture as domain‑centric, but the level of integration (e.g., independent four‑motor torque vectoring, DiSus‑P chassis system) implies tighter cross‑domain orchestration and higher internal bandwidth than earlier platforms.[14][4]

### 1.2 Current hardware stack and topology

#### 1.2.1 Domain controllers and central computing

BYD explicitly states that e‑Platform 3.0 uses an integrated, domain‑controlled E/E architecture with domain controllers for cockpit, body/vehicle control, powertrain, and intelligent driving. The company highlights that its vehicle control domain controller (body/VCU) and cockpit domain controller are in mass production, and that the 8‑in‑1 electric drive module integrates components such as the Vehicle Control Unit (VCU), Battery Management System (BMS), Power Distribution Unit, drive motor, motor controller, and onboard charger, effectively acting as a tightly coupled power domain controller.[12][3]

Yangwang U8’s intelligent driving stack uses an NVIDIA DRIVE Orin processor as the main ADAS/AD compute, with up to 508 TOPS specified, coupled with 38 high‑precision sensing components. This Orin‑based domain controller functions as a central compute node for perception, planning, and some chassis control coordination, surrounded by subordinate MCUs and inverters in the e⁴ platform’s four independent motor drives. For ADAS in volume segments, BYD uses Horizon Robotics’ Journey 5/6 AI processors, which deliver up to 128 TOPS per chip and are targeted at BEV perception and integrated driving‑and‑parking solutions.[15][7][4][14]

#### 1.2.2 Interconnects: Ethernet, CAN‑FD, LIN

BYD does not publish a full E/E topology, but several indicators suggest a hybrid network: the presence of high‑performance ADAS compute (Orin, Journey 5/6) and domain controllers implies the use of automotive Ethernet for camera and radar data, while lower‑level ECUs continue to communicate via CAN/CAN‑FD and LIN for cost‑sensitive actuators. References to “intelligent domain control architecture” and “rich cloud‑based application ecosystem” in e‑Platform 3.0 marketing, combined with OTA capabilities for the full vehicle, are consistent with an Ethernet‑backbone plus gateway pattern, where central/domain controllers and telematics/IVI connect over Ethernet, and legacy buses hang off zonal or domain gateways.[15][6][12][3]

Public technical analyses of modern zonal architectures show that domain controllers typically aggregate sensors and actuators via CAN‑FD and LIN sub‑buses, while high‑bandwidth data (multi‑camera, LiDAR) is carried via 100/1000BASE‑T1 Ethernet and switched across a central backbone. BYD’s use of multi‑LiDAR “God’s Eye A/B” variants and multi‑camera BEV solutions for Horizon Journey 5 strongly implies such an Ethernet‑centric high‑speed network inside the intelligent driving domain controller, but detailed bus speeds (10G vs 1G) are not disclosed.[16][17]

#### 1.2.3 Zonal gateways vs. domain controllers

Available material frames BYD’s current production architecture as domain‑centralized rather than fully zonal: the primary organizing principle is functional (powertrain, body, cockpit, ADAS) rather than pure physical zones. However, the four‑motor e⁴ platform and DiSus chassis control for Yangwang effectively act as zonal control of each wheel corner with a high‑bandwidth central coordinator, which is conceptually close to vertical zonal integration.[1][12][4][14]

For mainstream e‑Platform 3.0 vehicles (e.g., Dolphin, Atto 3, Seal), there is no explicit evidence of dedicated zonal controllers per physical zone; rather, domain controllers likely host multiple CAN gateways to front/rear/body segments. BYD is expected to gradually introduce more zonal partitioning in future evolutions (e‑Platform 4.0/4.x) as Ethernet penetration grows and wiring optimization becomes necessary to support even higher ADAS sensor density.

### 1.3 Supply chain and Tier‑1 / in‑house split

BYD’s FinDreams sub‑brand was created to supply EV parts including powertrain, batteries, and key electronic controls. FinDreams Powertrain is identified as the supplier of BYD’s electronic control systems, including inverters and controllers, and has achieved market‑leading share in electronic control systems in China. This indicates that many core powertrain controllers (drive inverters, hybrid supervisory controllers) are fully in‑house.[18][2]

On the ADAS compute side, BYD partners with Horizon Robotics (Journey 3/5/6) and NVIDIA (Orin) for high‑performance SoCs, while collaborating with software partners such as Momenta, Baidu, DJI, Holomatic, and Sensetime for algorithms and full‑stack AD solutions. For connectivity and 5G telematics, BYD has partnered with Huawei on at least some early models like the Han, leveraging Huawei’s MDC intelligent driving platform and Kirin‑based smart cockpit platform, although Huawei is positioned more as an incremental component supplier than a fully integrated Tier‑1.[7][19][6][4]

In the cockpit domain, BYD’s DiLink IVI stack runs on Qualcomm Snapdragon SoCs (e.g., 665, 690, 778G/782G across multiple chipset “generations”), indicating reliance on Qualcomm for infotainment hardware, but BYD develops its own DiLink software layer and integration. This blend of in‑house domain controllers, BYD OS, and external SoCs positions BYD closer to a “system integrator plus internal Tier‑1” model than a traditional OEM dependent on external Tier‑1s for complete ECUs.[20]

### 1.4 Controller vs. central processor strategy and software distribution

In e‑Platform 3.0, a significant portion of vehicle software has been migrated from scattered ECUs into a small set of domain controllers and the BYD OS layer. BYD explicitly states that BYD OS decouples hardware and software and runs atop the domain controllers, enabling higher‑level intelligent driving and OTA for “the whole vehicle.” However, actuator‑proximal control (brakes, steering, power distribution) still requires local MCUs running safety RTOS or Classic AUTOSAR‑style stacks, especially for ASIL‑D paths.[21][3][1]

For mainstream Ocean/Dynasty vehicles, the majority of new feature logic (infotainment, connectivity, higher‑level vehicle functions) appears to run in the cockpit domain controller (Android/DiLink) and vehicle control domain controller (BYD OS), with existing body and powertrain MCUs providing lower‑level deterministic control. In premium Yangwang/Fangchengbao products, a greater share of perception/planning software is centralized on Orin or high‑end Journey chips, while local controllers manage actuation (e⁴ motors, air suspension, steering actuators) under supervisory commands.[13][4][14][3]

Given typical patterns in domain‑centralized architectures and the degree of integration BYD claims, it is reasonable to infer that more than half of application‑level logic and feature software (by function count and update frequency) now runs on central/domain compute, while a smaller but safety‑critical portion runs on edge MCUs; the exact percentage is not disclosed, but the company’s emphasis on full‑vehicle OTA and BYD OS suggests a steadily increasing central share over time.[12][3]

***

## 2. Software Architecture, Standards, and OS Strategy

### 2.1 BYD OS, DiLink, and base operating systems

BYD repeatedly references an in‑house “BYD OS” as the vehicle operating system for e‑Platform 3.0, positioned as a decoupling layer between hardware and software that supports high‑level intelligent driving and cloud‑based application ecosystems. Corporate descriptions from BYD India and global marketing emphasize that the domain controllers plus BYD OS together “facilitate high levels of intelligent driving” and “achieve constant self‑evolution of the whole vehicle” via OTA.[21][13][1][12]

For the cockpit domain, BYD’s DiLink system is documented as an Android‑based IVI platform running on Qualcomm Snapdragon chipsets (e.g., 665, 690, 778G/782G), with Android 10 and customized UI layers (DiLink 3.0–5.0). DiLink supports a range of apps, 5G Internet of Vehicles, voice control, and OTA updates for the head unit, and it also participates in vehicle‑wide OTA campaigns as part of the BYD OTA ecosystem.[22][23][20]

On safety‑critical domains (powertrain, body, ADAS), BYD leverages safety‑grade RTOS and POSIX OSes indirectly via its chosen SoCs. NVIDIA DRIVE Orin stacks typically use QNX OS for Safety (and/or QNX Hypervisor) together with Linux‑based components, and community reports suggest that BYD’s Orin‑based systems use QNX at least in part as the safety foundation. With Horizon Robotics’ Journey chips, the base OS is generally a Linux or RTOS depending on SKU, with AUTOSAR Adaptive‑like middleware on top for ADAS applications, although BYD brands the higher‑level platform as BOS/BYD OS.[24][5][6][7]

To summarize the OS layering:

- Cockpit: Android‑based DiLink on Qualcomm SoCs (Android 10, BYD UI; no direct functional safety responsibilities).[23][20]
- Intelligent driving domain: Linux/QNX‑based stacks (from NVIDIA/Horizon) plus BYD‑developed BOS/ADAS framework.[5][6][4]
- Powertrain/body: Safety MCUs running RTOS or AUTOSAR Classic‑style OS for deterministic actuation, orchestrated by a vehicle control domain that interfaces with BYD OS.[2][3]

### 2.2 Virtualization, hypervisors, and vECUs

There is no explicit public description of BYD’s hypervisor deployment, but the presence of combined mixed‑criticality workloads on Orin and high‑end ADAS SoCs, plus typical patterns in the industry, strongly imply use of hypervisors and hardware partitioning. NVIDIA DRIVE Orin platforms commonly use QNX Hypervisor and/or other hypervisors to partition safety and non‑safety workloads and to support ISO 26262 certification, and QNX specifically promotes safety partitions combined with AUTOSAR Adaptive and POSIX workloads.[25][24]

Given that Yangwang U8’s Orin‑based intelligent driving domain integrates both high‑level ADAS algorithms and safety‑relevant chassis coordination, it is highly likely that BYD uses time‑ and space‑partitioned hypervisor configurations, with safety partitions hosting ASIL‑B/D functions and non‑safety partitions hosting comfort and data‑logging tasks. This is consistent with industry patterns where vECUs are instantiated within hypervisor partitions to host different functional stacks (e.g., AUTOSAR Adaptive apps, ROS‑derived perception stacks, vendor SDKs) alongside each other.[26][27]

For mainstream e‑Platform 3.0 vehicles using Horizon Journey 5/6, public press focuses on the BEV perception solution and DiPilot features rather than the exact virtualization layer; however, Horizon’s BPU architecture and typical ADAS deployments also rely on containerization or lightweight virtualization within Linux to run multiple perception/planning applications concurrently. Overall, BYD’s virtualization strategy appears pragmatic: heavy use of vendor‑supplied hypervisor/partitioning mechanisms on high‑end SoCs, with less emphasis on a unified vECU abstraction across all domains.[7][15]

### 2.3 Middleware and SOA framework

BYD’s e‑Platform 3.0 and BYD OS are marketed as enabling a “rich cloud‑based application ecosystem” and “standard interfaces available to the entire industry,” which strongly suggests a service‑oriented design where applications communicate via services rather than direct signal wiring. Public material, however, does not brand a specific SOA framework (such as AUTOSAR Adaptive or SOAFEE) by name.[28][3]

General industry practice for SDV‑style stacks is to use service‑oriented middleware (DDS, SOME/IP, or similar) running over Ethernet between high‑performance domain controllers and central compute, while Classic AUTOSAR remains on CAN‑based MCUs. Chinese technical presentations on intelligent driving domain controllers show AUTOSAR Classic on MCUs, Adaptive‑like stacks on POSIX OS (Linux/QNX), and DDS‑based data‑centric communication inside the controller and across the vehicle, and these patterns are likely mirrored in BYD’s intelligent driving domain given its use of similar SoCs and algorithms.[29][30][26]

Taken together, BYD’s SOA architecture can be characterized as:

- In‑house BYD OS providing platform services and standardized interfaces across domain controllers.[1][21]
- Vendor middleware (NVIDIA/Horizon) plus potential DDS‑style communication for ADAS data buses within intelligent driving domains.[30][6][7]
- Partial adoption of standardized SOA patterns (e.g., AUTOSAR Adaptive‑like APIs, POSIX, DDS), but wrapped under BYD branding rather than exposed as generic AUTOSAR/SOAFEE.

This is closer to Tesla’s proprietary stack than to a pure off‑the‑shelf AUTOSAR implementation, but BYD appears more heterogeneous and vendor‑dependent than Tesla’s fully in‑house microservices and event buses.

### 2.4 Standards posture: AUTOSAR (Classic/Adaptive) and SOAFEE

There is no official BYD statement publicly committing to AUTOSAR Adaptive or SOAFEE memberships, but the use of safety MCUs and intelligent driving SoCs makes it almost certain that AUTOSAR Classic (or AUTOSAR‑inspired architecture) is used in powertrain and chassis MCUs, and that AUTOSAR Adaptive‑compatible middleware is used on POSIX OSes in high‑performance domains. Suppliers such as Elektrobit (EB) provide Adaptive AUTOSAR stacks for QNX and Linux, and these are widely deployed in Chinese ADAS controllers; BYD could be using such stacks under BYD OS/BOS branding.[31][32][30][25]

SOAFEE (Scalable Open Architecture for Embedded Edge) primarily targets Arm‑based centralized compute with cloud‑native toolchains. There is no direct evidence that BYD officially adheres to SOAFEE, and Chinese OEMs generally focus more on proprietary or domestically driven frameworks combined with AUTOSAR‑derived principles and containerization. BYD’s emphasis on BYD OS and its own cloud ecosystem suggests an independent trajectory rather than explicit SOAFEE alignment.[33][26]

### 2.5 Functional safety and mixed‑criticality isolation

To achieve ISO 26262 compliance at ASIL‑B/D while mixing safety‑critical and infotainment or non‑safety applications on shared hardware, BYD relies on several layers:

- Safety‑certified MCUs and RTOS/AUTOSAR Classic for core motion control (braking, steering, torque) within the powertrain and body domains.[30][2]
- Safety‑certified AI chips: Horizon Journey 5 has TÜV‑certified ISO 26262 ASIL‑B functional safety certification, enabling its use in safety‑relevant ADAS functions.[34]
- Safety‑grade OS and hypervisors: QNX OS for Safety and QNX Hypervisor (or equivalents) provide time‑ and space‑partitioning, enforcing isolation between safety and non‑safety partitions on SoCs like Orin.[24][25]
- Architectural patterns where infotainment (Android/DiLink) remains on a separate SoC from motion control and safety‑relevant ADAS, avoiding direct mixing of ASIL‑D workloads with non‑safety UI processes.[20][23]

BYD’s mixed‑criticality strategy therefore looks conventional for the industry: strict partitioning of Android IVI from safety domains, use of hypervisors and partitions on central ADAS SoCs, and reliance on safety‑certified RTOS and MCUs for ultimate actuation control. The BYD OS layer coordinates feature logic and OTA across these domains without collapsing their safety boundaries into a single shared environment.[12][1]

***

## 3. Autonomous Driving, AI, and Silicon Strategy

### 3.1 In‑house vs. partnered ADAS software development

BYD has significantly increased investment in ADAS since around 2021, creating a 60/40 joint venture with Momenta focused on autonomous driving algorithms and reaching further cooperation with AI chip company Horizon Robotics for compute platforms. Reports indicate that BYD uses separate solutions for different segments and features: Journey 5 for highway NOA and high‑level city lane‑centering (LCC) in models like the Han, Tang, and Song, while also deploying NVIDIA Orin for more advanced intelligent driving in premium models.[6][4]

Beyond Momenta and Horizon, BYD collaborates with DJI, Holomatic, Sensetime, and Baidu on various ADAS and smart driving functions, creating a diversified partner ecosystem instead of a single monolithic AD stack. At the same time, BYD has built an internal “BOS” or BYD OS for the ADAS platform, and presentations from BYD engineers describe internal computation platforms for vehicles like Yangwang U8 and Denza N7, including massive data pipelines for ADAS training and validation.[6]

This leads to a split where:

- Core platform software (BYD OS/BOS, data platforms, integration, calibration, and safety cases) is developed in‑house.[21][12]
- Perception and NOA algorithms are co‑developed or licensed from partners such as Momenta and Baidu, tuned and integrated by BYD to run on Horizon Journey or Orin platforms.[7][6]
- ADAS feature packaging (DiPilot, “God’s Eye” ADAS tiers) and UI integration are productized internally.

### 3.2 Silicon landscape across vehicle tiers

BYD’s current and near‑term AI and ADAS compute portfolio spans multiple vendors and tiers:

- Horizon Robotics Journey series: BYD announced in 2022 that it would equip some 2023 models with Horizon’s Journey 5 chip, with BYD’s proprietary BEV perception solution entering mass production and expanding across models over time. Journey 5 offers up to 128 TOPS of AI compute, supports 16 camera channels, and has ISO 26262 ASIL‑B certification, making it suitable for L2+/L3‑ish ADAS.[34][7]
- NVIDIA DRIVE Orin: Yangwang U8 Premium Edition uses an NVIDIA DRIVE Orin intelligent driving processor with up to 508 TOPS, combined with 38 high‑precision sensors, enabling high‑end ADAS and potential higher automation levels in the future.[4][14]
- BYD’s DiPilot “God’s Eye” tiers: Public descriptions of the “God’s Eye” ADAS system describe three compute tiers mapped to DiPilot 100, 300, and 600 platforms, delivering 100, 300, and 600 TOPS respectively across standard BYD models, Denza, and Yangwang, with some variants using up to three LiDAR sensors. DiPilot 100 targets standard models like Seagull; DiPilot 300 targets mid‑range Dynasty/Ocean vehicles; DiPilot 600 targets Yangwang’s luxury segment.[8][35]
- Qualcomm Snapdragon for IVI: Multiple BYD head units use Snapdragon 665, 690, and 778G/782G SoCs, which provide GPU and NPU capabilities for local voice, UI, and some perception pre‑processing in the cockpit, though not primary ADAS compute.[20]

In earlier collaborations with Huawei, BYD’s Han model integrated Huawei’s 5G connectivity and MDC intelligent driving platform for early intelligent driving trials, though subsequent ADAS scaling has focused more on Horizon and in‑house/BYD‑Momenta stacks.[19]

### 3.3 AI ecosystem and partners

BYD’s AI ecosystem is broad and multi‑layered:

- Algorithm partners: Momenta (NOA and city driving algorithms), Baidu (Apollo‑derived functionalities), DJI (parking and low‑speed functions), Holomatic, and Sensetime (perception and vision AI) contribute to different parts of the ADAS stack.[6]
- Compute partners: Horizon Robotics (Journey 3/5/6), NVIDIA (Orin), and previously Huawei (MDC, Kirin) provide heterogeneous hardware platforms for different vehicle tiers.[19][34][4]
- Data and training infrastructure: BYD has built internal computation platforms and data centers for its ADAS, as referenced in presentations that show U8 and N7 data volumes and training pipelines, indicating that large‑scale data collection and labeling are increasingly in‑house even when algorithms are co‑developed.[6]

For LLMs and generative AI in vehicles, there is little explicit information. However, given China’s broader cloud and AI ecosystem, BYD is likely to cooperate with domestic cloud providers and AI companies (Baidu, Alibaba, Tencent) for voice assistants and in‑car conversational AI, integrated into the DiLink stack. This resembles competitive offerings from Xiaomi and other Chinese OEMs, but BYD’s public communication focuses more on driving intelligence (God’s Eye, DiPilot) than on LLM branding.

### 3.4 “God’s Eye” ADAS roadmap

“God’s Eye” is BYD’s branded ADAS family, recently announced for rollout across the entire vehicle lineup, including budget models like Seagull. It is offered in three main tiers: God’s Eye C (entry‑level, three‑camera DiPilot 100), God’s Eye B (DiPilot 300 with up to 300 TOPS and highway/city autonomous driving support), and God’s Eye A (DiPilot 600 with 600 TOPS, multiple LiDARs, and high‑end intelligent driving for Yangwang).[35][8]

The roadmap indicates that God’s Eye A/B target both urban and highway environments, while God’s Eye C initially targets highway and later receives city capabilities via OTA. A total of 21 models across Dynasty, Ocean, Denza, and Yangwang will be equipped, pointing to deep penetration of high‑level ADAS into the mass market. With BYD’s strong data collection infrastructure and partnerships, this positions the company as a leading ADAS deployer in terms of fleet learning potential.[8]

***

## 4. Connectivity, OTA, and Connected‑Car Infrastructure

### 4.1 Ground‑up connectivity stack

BYD’s connectivity strategy combines built‑in cellular connectivity, an internal OS and telematics stack, and cloud platforms. Connected BYD vehicles are equipped with built‑in 4G/5G SIM or eSIM modules, enabling independent data connectivity without relying on the user’s phone; this connection is used for real‑time data upload to BYD’s backend cloud and for remote control via the BYD app.[36]

The DiLink intelligent connectivity system acts as the in‑vehicle client for telematics, combining HMI, vehicle status monitoring, and app integration, with frequent sensor and status updates flowing to the cloud (reports mention updates every tens of seconds for some diagnostics data). BYD’s connected car platforms are being used in pilot programs such as digital road user charging (RUC) in New Zealand, where BYD integrates its in‑vehicle telematics with Compass IoT’s road intelligence platform to provide fleet‑level data.[37][36][23]

Connectivity to cloud services is described as an enabler for “vehicle–road–cloud” cooperation in early 5G collaborations with Huawei, where BYD Han was the first 5G‑equipped production car, leveraging 4G, 5G, and V2X technologies for vehicle‑to‑infrastructure and vehicle‑to‑vehicle communication, reinforced by cloud‑based services for managing connected fleets.[19]

### 4.2 OTA update mechanisms and technology

e‑Platform 3.0 and BYD OS explicitly support continuous and automatic updates for the entire vehicle, not just the IVI system. The BYD OTA system is capable of updating multiple ECUs (including vehicle control and ADAS) and is tightly integrated with DiLink for user interaction, scheduling, and version management.[3][22][1][12][20]

While BYD does not fully document its OTA protocol stack, industry practice and generic SOTA/FOTA architectures suggest it uses delta‑based firmware updates, secure partitions, and transactional flashing. OTA reference architectures highlight delta file‑based updates over secure channels, with components including a device manager, download manager, and update agent that combine current images with received deltas and enforce rollback on failure, and BYD’s reference to whole‑vehicle OTA and long update times (tens of minutes) for Atto 3 campaigns align with such mechanisms.[38][39][22]

For OTA security, BYD likely uses standard practices such as TLS‑secured downloads, signed firmware, and secure boot, especially where ISO 26262 and UNECE R155/R156 compliance is required. Public Atto 3 OTA information indicates structured versioning (OTA 1.0.1 and 1.0.2) and controlled rollout over 10 days, consistent with staged deployment and rollback capabilities.[22]

### 4.3 OTA cadence and scope

Region‑specific BYD communications suggest that OTA updates occur “every few months,” with major updates accompanied by email notifications and release notes. Atto 3’s initial OTA took about 57 minutes, followed by a secondary OTA within 72 hours taking around 27 minutes, indicating multi‑phase updates for both the head unit and other ECUs.[10][22]

A 2026 report notes that BYD’s Ocean and Dynasty series collectively received approximately 200 OTA updates in 2025, leading the industry and significantly outpacing legacy OEMs like Toyota and Volkswagen. This figure likely counts individual rollout campaigns or ECU‑level update events, demonstrating both high iteration velocity and a granular, modular OTA infrastructure.[9]

The scope of OTA spans:

- Infotainment (DiLink apps, UI, voice assistants).[23][20]
- ADAS and intelligent driving feature updates, including NOA and “God’s Eye” enhancements.[35][8]
- Vehicle control and BMS updates for efficiency, range, and drivability, enabled by the domain‑controlled E/E architecture.[3][12]

***

## 5. Organizational and Market Strategy

### 5.1 Software‑defined organizational structure

Public disclosures about BYD’s internal organization are limited, but several indicators point to a structure aligned with SDV principles:

- Dedicated OS and platform teams: The development of BYD OS and DiLink implies dedicated internal software platform organizations responsible for cross‑vehicle software, domain controllers, and OTA.[12][3][21]
- ADAS and intelligent driving units: Formation of a JV with Momenta, extensive partnerships with AI firms, and building internal computation platforms suggest a sizable ADAS engineering organization with data, platform, algorithm, and validation sub‑teams.[6]
- Vertical product lines: FinDreams subsidiaries (batteries, powertrain, electronics) act as internal Tier‑1s, with their own engineering and manufacturing organizations delivering standardized components and controllers across vehicle lines.[2]

Industry analyses of Chinese “new‑energy OEMs” emphasize that leaders like BYD increasingly adopt platform organizations for software and E/E, where cross‑domain features (connectivity, OTA, ADAS) are built centrally and reused across Ocean, Dynasty, Denza, and Yangwang brands. BYD’s ability to roll out the “God’s Eye” ADAS across 21 models and to deliver hundreds of OTA updates in a year supports the existence of such centralized software platform teams.[9][8]

### 5.2 Cost and velocity competitiveness

BYD’s cost and speed advantages stem from four main pillars:

1. **Extreme vertical integration**: BYD designs and manufactures its own batteries (Blade), power electronics, electric motors, domain controllers, and OS platform (BYD OS), reducing dependency on external Tier‑1s and capturing margin along the value chain.[1][2][3]
2. **Standardized e‑Platform 3.0 base**: Most Ocean and Dynasty models are built on e‑Platform 3.0, sharing the 8‑in‑1 drive unit, Blade Battery integration, and domain‑controlled E/E architecture, enabling reuse of hardware and software across a high volume of vehicles.[13][12]
3. **Aggressive OTA and software iteration**: BYD’s high OTA cadence (approx. 200 updates per year across major series) allows it to ship hardware earlier and progressively refine software, shortening time‑to‑market and extending product life without costly hardware refreshes.[10][9]
4. **Chinese ecosystem leverage**: Collaborations with Horizon, Momenta, Baidu, DJI, and others give BYD access to leading‑edge ADAS algorithms and silicon at domestic cost structures, while keeping core integration and productization in‑house.[8][7][6]

Compared with legacy OEMs still transitioning from ECU‑centric, supplier‑driven architectures, BYD’s integrated platform, unified OS, and rapid OTA cycles translate directly into lower development and integration costs per feature and faster rollout of intelligent functions across a wide price spectrum.

***

## 6. Comparative Syntheses and Strategic Assessment

### 6.1 Evolution matrix: BYD’s E/E and software stack over generations

| Generation | Approx. period | E/E architecture | Compute / controllers | Software & OS | OTA & connectivity |
|-----------|----------------|------------------|-----------------------|---------------|--------------------|
| Early distributed | Pre‑2018 | Distributed ECUs connected via CAN/LIN; limited domain logic | Multiple small ECUs for powertrain, body, chassis; basic infotainment MCU | Vendor RTOS, limited BYD software; basic HMI | Limited telematics, no full‑vehicle OTA |
| Proto‑domain (e‑Platform 2.x) | ~2018–2020 | Emerging domain logic for powertrain/body; still ECU‑heavy | Power electronics and some controllers in‑house via FinDreams | Growing BYD software in body and powertrain; early DiLink | Connected IVI, app integration, partial OTA for head unit |
| Domain‑centralized (e‑Platform 3.0) | 2021–present | Integrated, domain‑controlled E/E with four domain controllers: smart powertrain, vehicle control, smart cockpit, intelligent driving | 8‑in‑1 drive unit (VCU+BMS+PDU+motor+MCU+charger), cockpit and body domain controllers, Horizon Journey 5/6 for ADAS | BYD OS decouples HW/SW; DiLink on Android; BOS for intelligent driving; AUTOSAR‑style MCUs beneath | Full‑vehicle OTA; frequent updates every few months, with ~200 OTA events in 2025 across Ocean/Dynasty; 4G/5G connectivity and app remote control[12][3][9][13][10] |
| High‑centralization premium (Yangwang/Fangchengbao) | 2023–present | Domain‑centralized with strong cross‑domain coordination (e⁴ platform, DiSus‑P chassis); trending toward zonal behavior | NVIDIA Orin 508 TOPS for ADAS; four independent motor drives; advanced chassis controllers | BYD OS/BOS orchestrating high‑end ADAS, torque vectoring, and body control; more sophisticated SOA on Orin | High‑end OTA for ADAS and chassis; early V2X / 5G features via Huawei and others[4][14][19] |

### 6.2 SDV litmus test: How “software‑defined” is BYD vs. Tesla and Xiaomi?

Using a pragmatic SDV litmus anchored in compute centralization, service‑oriented software, OTA depth, and organizational alignment:

- **Compute centralization**: Tesla’s latest platforms approach a single central computer with a small number of satellite ECUs, whereas BYD is presently domain‑centralized with some premium models edging toward zonal behavior and powerful ADAS compute, and Xiaomi’s early vehicles are architected as high‑centralization domain/zonal designs leveraging Qualcomm/NVIDIA with deep Android integration. BYD lags Tesla in pure centralization but is likely comparable to Xiaomi’s first‑gen in practice.[26][4]
- **Software architecture**: Tesla runs a proprietary, tightly integrated microservices/event bus architecture; BYD uses BYD OS plus vendor stacks and AUTOSAR‑style components, with less visible unification but still strong platform reuse across e‑Platform 3.0 vehicles. Xiaomi positions its car OS as tightly integrated with its broader ecosystem; BYD’s DiLink is more automotive‑specific and less consumer‑ecosystem‑centric.[3][12]
- **OTA depth and frequency**: BYD’s approx. 200 OTA updates in 2025 across Ocean/Dynasty are on par with or exceeding most global OEMs, although Tesla has long been the benchmark for continuous deployment in vehicles. Xiaomi is early but aims for frequent OTA; BYD is already at scale.[9]
- **ADAS/AI integration**: Tesla uses predominantly in‑house ADAS/AD software and custom chips (e.g., FSD computer), whereas BYD relies on Horizon, NVIDIA, and a web of algorithm partners, integrating them via BYD OS/BOS; Xiaomi, similarly, relies on Qualcomm/NVIDIA and Chinese AI ecosystems.[4][7][6]

Conclusion of the litmus test: BYD is clearly beyond “feature‑defined” and into the SDV spectrum, with domain‑centralized hardware, a unifying OS, and high OTA cadence, but it is not as monolithically software‑first as Tesla. It is roughly in the same SDV maturity band as leading Chinese NEV peers (Xiaomi, NIO, XPeng), with some advantages in scale and OTA cadence but less aggressive public positioning around centralized compute and L3 autonomy.

### 6.3 Key vehicle archetypes and architecture deployment

BYD deploys its architectures across distinct product archetypes:

- **Dynasty series (Han, Tang, Qin, Song, Yuan)**: Mostly built on e‑Platform 3.0 or its hybrid equivalents, with domain‑controlled E/E, BYD OS, DiLink IVI, and increasing penetration of Journey‑based ADAS and mid‑tier “God’s Eye” B/C packages.[13][8][12]
- **Ocean series (Dolphin, Seal, Atto 3 / Yuan Plus)**: Also based on e‑Platform 3.0, emphasizing efficiency and cost; domain controllers and BYD OS are standard, with DiLink and OTA; “God’s Eye” C/B offers affordable ADAS even on lower‑priced models like Seagull.[8][13][12]
- **Denza**: Positioned as premium, using enhanced versions of e‑Platform 3.x with richer ADAS and cockpit features, likely with Journey 5/6 and mid‑ to high‑tier God’s Eye B packages, and more sophisticated service integration.[8][6]
- **Yangwang (U8, others) and Fangchengbao**: High‑end and off‑road performance brands, built on architectures combining e‑Platform 4/e⁴ with advanced chassis (DiSus) and Orin‑based intelligent driving domains; here, compute centralization and cross‑domain control are most advanced, with God’s Eye A/DiPilot 600 and multi‑LiDAR setups.[14][4][8]

This portfolio demonstrates a common hardware/software foundation (e‑Platform 3.x + BYD OS + DiLink + OTA + ADAS platforms) scaled across price points, with premium brands adding higher compute density and more advanced ADAS.

### 6.4 BYD’s differentiation playbook: speed and cost advantages

BYD’s global competitiveness in SDV/AI‑DV hinges on several differentiators:

- **Unified EV platform and E/E**: e‑Platform 3.0, with its 8‑in‑1 drive unit, Blade Battery integration, and domain‑controlled E/E architecture, reduces parts count, wiring complexity, and integration cost while enabling full‑vehicle OTA and shared software assets.[12][3]
- **Vertical integration with FinDreams**: Internal control over powertrain, batteries, and electronic controls (via FinDreams) reduces supplier margins and optimizes engineering cycles; FinDreams Powertrain already supplies electronic controls externally, underscoring BYD’s role as a Tier‑1‑like entity.[18][2]
- **BYD OS and DiLink as reusable software platforms**: The same OS and IVI stack underpin Dynasty, Ocean, Denza, and Yangwang, making feature development and bug fixes amortizable across millions of vehicles, and enabling rapid OTA rollout of new functions.[13][1]
- **Ecosystem‑driven ADAS**: BYD’s choice to integrate multiple Chinese AI partners allows it to cherry‑pick best‑of‑breed perception and NOA stacks and to hedge against single‑vendor lock‑in, while still owning the integration, validation, and feature definition.[8][6]
- **High OTA cadence and data feedback loops**: The combination of built‑in 4G/5G connectivity, frequent OTA campaigns, and large fleet sizes generates fast feedback for software, ADAS models, and UX tuning, shortening iteration cycles and providing a data moat over slower competitors.[36][9]

Relative to Tesla, BYD’s key advantages are cost structure and local ecosystem depth in China; relative to emerging players like Xiaomi, its advantages are scale, manufacturing depth, and a mature, already‑deployed OTA and ADAS architecture. The main strategic risk is architectural heterogeneity (multiple SoCs, multiple ADAS partners), which may complicate long‑term software unification compared with Tesla’s single‑stack approach.

# Reference


[^1]: https://linkinghub.elsevier.com/retrieve/pii/S2214750024000441

[^2]: https://aslopubs.onlinelibrary.wiley.com/doi/10.1002/lom3.10549

[^3]: https://sol.sbc.org.br/index.php/webmedia_estendido/article/view/38180

[^4]: https://e-journal.unair.ac.id/JESTT/article/view/39171

[^5]: https://www.mdpi.com/2227-7080/13/11/496

[^6]: http://biorxiv.org/lookup/doi/10.64898/2026.02.06.704361

[^7]: https://pubs.acs.org/doi/10.1021/acsinfecdis.6c00136

[^8]: https://journal.unesa.ac.id/index.php/jetis/article/view/38513

[^9]: https://www.byd.com/eu/technology/byd-e-platform-3

[^10]: https://autonews.autoua.net/en/63697-byd-claims-80-crash-drop-from-its-adasbut-the-system-wont-leave-china.html

[^11]: https://en.byd.com/news/byd-launches-e-platform-3-0-with-the-new-concept-car-ocean-x/

[^12]: https://www.drive.com.au/news/byd-launches-driver-assistance-tech-with-ai-integration-including-for-its-cheapest-models/

[^13]: https://bydeurope.com/article/363

[^14]: https://www.byduae.ae/en/news/adas-features-in-byd/

[^15]: https://electronsx.com/ev-platforms-byd.html

[^16]: https://www.byd.com/za/news-list/byd-dipilot-intelligent-driving-assistance

[^17]: https://bydautoindia.com/eplatform3.0

[^18]: https://kr-asia.com/byd-says-it-will-stay-the-course-on-assisted-and-smart-driving

[^19]: https://www.youtube.com/watch?v=VDAwMYlTluk

[^20]: https://cleantechnica.com/2025/02/17/byd-gods-eye-brings-adas-to-the-masses/

[^21]: https://www.batterydesign.net/byd-e-platform-3-0/

[^22]: https://www.electrive.com/2025/07/10/byd-adds-level-4-autonomous-parking-to-adas/

[^23]: https://www.marklines.com/en/report/rep2692_202406


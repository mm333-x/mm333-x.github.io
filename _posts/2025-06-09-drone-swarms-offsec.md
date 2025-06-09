---
title: Drone Swarm Cybersecurity from Offensive Perspective (Swarm Hijacking by Sybil Attack - SHSA)
published: 2024-07-20
description: '"This research aims to demonstrate, from an offensive perspective and through simple proof-of-concept experiments using the MAVLink protocol, why drone swarms must be designed with cybersecurity as a foundational principle. The concepts can be simulated using popular drone platforms such as PX4 SITL, Gazebo, AirSim, and Mission Planner.'
image: '/assets/drone_swarm.webp'
tags: ["Drone", "Drone Cybersecurity", "Drone Hacking"]
category: 'Drone Cybersecurity'
draft: false 
---
<!-- markdownlint-capture -->
<!-- markdownlint-disable --> 
>This research is intended for testing purposes only within mission simulator tools or strictly controlled physical environments. Use responsibly and ethically, this project is designed solely for educational and research purposes. Always adhere to legal and ethical hacking guidelines.
{: .prompt-danger }
> <!-- markdownlint-restore -->

# Introduction

The mission of this research is to demonstrate, through simple proof-of-concept research utilizing the **MAVLink** protocol how drone swarms must be designed with **cybersecurity as a foundational principle**:

- Encrypted communication channels  
- Strong authentication and message signing  
- Effective rogue drone detection  
- Reliable collision avoidance mechanisms  
- Secure leader selection algorithms  
- Resilient mission continuity strategies  

This research underscores that without these essential security measures, drone swarms remain vulnerable to attacks that can undermine their operational integrity and safety.

---

**Why MAVLink?**

This research uses **MAVLink** due to its **prevalence in both academia and commercial development**. For the demonstration and proof-of-concept, we use **MAVLink**, an open-source UAV communication protocol, to simulate:

- Swarm coordination  
- Leader election  
- Hijacking scenarios  

The research paper is based on the **official MAVLink documentation**, different **drone simulators**, and the **Python programming language**.

---

**Proof-of-Concept Scenario**
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> In this proof-of-concept scenario, the drone swarm elects its leader based on **battery health**, which exposes vulnerabilities that adversaries can exploit to **manipulate swarm behavior**.
{: .prompt-info }
> <!-- markdownlint-restore -->

**Mesh Networking**

Drone swarms commonly rely on **mesh networking architectures** to maintain reliable and decentralized communication between swarm members. In a mesh network, each drone acts as a **node** that can communicate directly with its neighbors and relay messages across the network. 

This architecture allows drones to form an **adaptive, self-healing communication grid** that does not depend on a centralized base station or constant access to ground control systems.

**Benefits of Mesh Networks in Swarm Operations**

- **Redundancy and Fault Tolerance**:  
  If one or more drones fail or lose connectivity, the network can dynamically reroute data through alternate paths, maintaining operational integrity.

- **Scalability**:  
  New drones can be added to the swarm with minimal reconfiguration, and the network adjusts automatically to accommodate changes in formation or node availability.

- **Mobility Support**:  
  As drones move, the mesh topology evolves to preserve communication links without interrupting mission-critical tasks.

- **Decentralized Coordination**:  
  Mesh networks support distributed decision-making protocols, such as leader elections, consensus formation, and collaborative sensing.

![parrot the emu 2](/assets/mesh-drone-network.png)

Larger drone swarms often use a **partial mesh**, where drones connect only to nearby nodes within communication range. The network adapts dynamically as drones move, due to range limitations, radio interference, and power constraints. These properties make mesh networks a compelling solution for autonomous swarm missions, particularly in environments where infrastructure is limited or adversarial interference is expected.

However, this architecture also introduces unique security risks, such as susceptibility to **Sybil attacks**. This paper seeks to explore this risk through simulation and proof-of-concept implementations, highlighting the need for security-aware swarm design from the ground up.

**Leader Drone**

In drone swarm systems, the **leader drone** plays a central role in coordinating the actions of other swarm members. While some swarms operate in fully decentralized modes, many adopt a **leader-follower architecture** to simplify decision-making, reduce communication complexity, and optimize formation control.

The leader drone can handle various critical functions. It may coordinate the overall mission by determining the global trajectory or mission objective and distributing this information to follower drones. It often manages **formation control**, acting as the spatial anchor around which followers adjust their positions. Additionally, the leader can provide **synchronization cues**, triggering coordinated behaviors such as takeoff, landing, or turning. In some designs, the leader also serves as a **communication relay**, acting as the primary link between the swarm and the ground control station (GCS).

![parrot the emu 2](/assets/leader-drone.png)

**Leader Drone and Insecure Election**

Leader drones can be either predefined or elected dynamically. In the case of predefined or static leaders, a specific drone is hardcoded or manually designated to act as the leader before the mission begins. In contrast, dynamic leaders are chosen during operation based on real-time metrics such as GPS signal strength, battery life, trust scores, or computing power. Various leader election algorithms, including the Bully algorithm, RAFT consensus, and custom trust-weighted selection schemes, are used to ensure resilience and allow seamless leadership handover in the event of failure or compromise.

Because of its elevated role, the leader drone represents a high-value target in adversarial scenarios. If compromised, the leader can redirect the swarm, sabotage the mission, or cause collisions. Even more critically, if the leader election process itself is insecure, rogue drones may exploit it to impersonate or become the leader - this is a core focus of the research presented in this paper.

When leader election is required for purposes such as coordinating formation movement or managing communication with ground control stations, many swarm implementations rely on ad hoc or simplistic criteria for determining which drone should assume the leader role. Common approaches include selecting the drone with the highest battery health, the strongest GPS signal, or even the lowest or highest numerical ID.

Such leader selection processes can be exploited by malicious entities. A rogue drone injected into the swarm can falsely advertise favorable attributes, such as superior battery life or a high-priority ID, to game the election. In a fully connected mesh network, where every node can communicate with every other node, the rogue drone can quickly disseminate deceptive metrics and illegitimately position itself as the swarm leader. Once elected, the compromised leader gains the ability to manipulate formation trajectories, inject false navigation commands, isolate nodes from the group, or even steer the swarm into hazardous areas.

These inherent weaknesses in leader election mechanisms, especially within mesh-based swarms, highlight the urgent need for more secure approaches. Trust-based selection algorithms, cryptographically secure node authentication, and fallback leadership protocols must be integrated to mitigate the risks of leadership hijacking. The attack model proposed in this paper exploits these vulnerabilities to demonstrate the feasibility and consequences of leader impersonation in autonomous drone swarms.

# Swarm Hijacking by Sybil Attack (SHSA)

Swarm Hijacking by Sybil Attack refers to a form of takeover in drone swarms where a rogue drone injects itself into the swarm communication network by impersonating one or more legitimate nodes (drones). This enables the attacker to assume control roles (e.g., leader) and inject malicious commands, resulting in loss of coordination, disruption, or misdirection of the swarm.

### Sybil Attack

A Sybil attack is a type of security threat where a single entity forges multiple identities to gain disproportionate influence in a system. In a drone swarm, this means one rogue drone pretending to be multiple drones or claiming an identity like the leader.

### Swarm Hijacking

Swarm Hijacking refers to the unauthorized takeover or disruption of coordinated behavior in a multi-agent system (e.g., drone swarm). It can result in mission failure, segmentation, or unintended behavior.

# Attack Scenario

## Phase I - Physical Infiltration

For a rogue drone to perform a successful Swarm Hijacking via Sybil Attack, physical proximity to the drone swarm is a prerequisite. This is primarily due to the communication used in drone-to-drone networking so the rogue drone must be within wireless transmission range of the swarm's network (operational radius of the target swarm). This range depends on technology used, signal strength, antenna power, and environmental interference.

![parrot the emu 2](/assets/rogue-drone.png)

Once the rogue drone is deployed within the swarm’s operational radius, either by manual piloting or autonomous navigation, it initiates peer discovery and begins emulating the expected communication patterns of legitimate swarm members. By establishing direct links with nearby drones, it passively integrates into the mesh network. This marks the transition to **Phase II**, where active infiltration and manipulation of swarm behavior begin.

![parrot the emu 2](/assets/operating_radius.png)

## Phase II – Network Reconnaissance and Intelligence Gathering

Once the rogue drone successfully enters the operational radius of the target drone swarm, it proceeds to passively observe and analyze the swarm’s internal communication. This stage is critical for understanding the mesh network’s topology, discovering the communication protocols in use, and identifying potential opportunities for impersonation or command injection.

The rogue drone initiates promiscuous listening on the wireless spectrum used by the swarm. By capturing raw packets, it can reconstruct swarm communication patterns without generating any transmission. One of the rogue drone’s primary reconnaissance goals is to determine the current swarm leader, which is typically the node responsible for issuing movement commands to the rest of the swarm. Many swarm coordination implementations use a decentralized leader election algorithm based on heuristics, such as battery health, GPS stability, or ID precedence (see Insecure leader drone election).

In scenarios where the rogue drone is equipped with sufficient onboard computing resources and a compatible wireless adapter running in monitor mode as drone payload, the reconnaissance script can be executed directly on the drone. This enables autonomous infiltration of swarm communications, identification of leadership heuristics, and preparation for injection, all without ground station intervention. 

Although the rogue drone's flight controller may run a lightweight RTOS incapable of executing Python scripts, this limitation is overcome by integrating a payload computer with full OS support (e.g., Linux). This architecture allows the rogue drone to run advanced network reconnaissance and MAVLink manipulation tools, acting as an intelligent node in the attack pipeline.

In this proof-of-concept implementation, we utilize a Python script to passively scan wireless channels in monitor mode, identifying MAVLink traffic by detecting the protocol’s start byte (0xFE for MAVLink v1 or 0xFD for MAVLink v2). Information is extracted directly from packet payloads to construct a real-time swarm map. 

<!-- markdownlint-capture -->
<!-- markdownlint-disable --> 
>While not implemented in this proof of concept, monitoring the frequency and volume of messages sent by each drone could allow the system to infer hierarchical patterns, detect anomalies, and track leadership dynamics within the mesh swarm.
{: .prompt-tip }
> <!-- markdownlint-restore -->

![parrot the emu 2](/assets/python1.png)

![parrot the emu 2](/assets/python2.png)

In this phase, the system passively monitors MAVLink traffic over a 60-second window on a wireless interface in monitor mode. It extracts the IP address, UDP port, and battery status of each detected drone, enabling swarm mapping and energy-based leader inference.

![parrot the emu 2](/assets/python3.png)

Based on the gathered intelligence, we can conclude that the leader drone is likely 192.168.1.14, as it has the highest remaining battery level among all swarm nodes.

## Phase III – Drone Leader Hijacking via MAVLink Packet Injection

In this stage, the rogue drone begins actively injecting carefully crafted MAVLink packets. These packets emulate legitimate telemetry, heartbeat, or coordination messages exchanged by swarm members, allowing the rogue drone to manipulate swarm behavior without immediate detection. By mimicking or overriding decision-making signals, such as those used in decentralized leader election algorithms based on drone ID, battery level, or role assignments, the rogue drone exploits the lack of authentication and encryption in the mesh communication protocol to assert itself as the new swarm leader. This marks a critical escalation, where control of the swarm’s coordination logic is subtly hijacked from within.

![parrot the emu 2](/assets/python4.jpg)

This script iterates over a list of UDP endpoints corresponding to individual drones within the swarm. For each drone, it establishes a MAVLink connection using a forged system ID (source_system=255), then sends a single forged BATTERY_STATUS claiming a battery level of 99%. This approach aims to mislead each drone in the swarm into believing that a rogue entity has superior battery health, potentially influencing decentralized leader election or other swarm coordination algorithms that rely on battery telemetry.

<!-- markdownlint-capture -->
<!-- markdownlint-disable --> 
>Waiting for a heartbeat ensures that the connection to each drone is properly established and synchronized before sending commands, preventing lost or ignored messages. In real-life scenarios, attacker would perform this synchronization simultaneously across the swarm to efficiently inject commands without delay.
{: .prompt-tip }
> <!-- markdownlint-restore -->

The battery_status_send function includes several fields simulating battery telemetry. The voltage is set to 12000 millivolts (12V), representing a typical fully charged 3-cell LiPo battery (3 × 4.2V = 12.6V). The current is 2000 milliamps (2A), simulating the current being drawn. The temperature is set to 2500 (25.00°C), using MAVLink’s convention of temperature in centi-degrees. The voltages list can hold up to 10 cell voltages, but here only one is provided and the rest are padded with zeros. In real systems, power (P) is calculated as voltage (V) times current (I), so 12V × 2A = 24W, which can be useful in estimating energy consumption. The battery_remaining is a percentage value used in swarm logic for leader election, making it a key target for manipulation.

In the battery_status_send message, the id field represents the battery number (e.g., id=0 for the first battery). MAVLink supports reporting multiple batteries in a single system, which is useful for drones with redundant or extended power sources. In the script, id=0 indicates that the rogue drone is simulating data for its primary battery. If multiple batteries were present, each would have a separate ID and corresponding telemetry data. Accurately reporting or spoofing multiple batteries could further influence swarm decision-making in systems that consider total available energy across all power sources.

Following the successful execution of the battery spoofing attack by packet injection, the rogue drone assumes the leader role within the swarm, enabling the attacker to initiate **Phase IV** by issuing coordinated commands to manipulate swarm behavior.

![parrot the emu 2](/assets/leader_takeover.png)

## Phase IV – Swarm Command and Control Exploitation

In **Phase IV**, having successfully assumed the leader role within the drone swarm, the attacker gains the capability to directly influence and control other drones in the formation. This elevated position allows the adversary to manipulate swarm behavior with precision, enabling disruptive outcomes such as Denial of Flight (DoF), which prevents drones from initiating or safely continuing their flight operations, or Denial of Mission (DoM), which obstructs the swarm from completing its assigned tasks or objectives. These outcomes are achieved through advanced cyber-physical attack techniques targeting the swarm’s command and control infrastructure.
One particularly potent attack vector is the Swarm-Level Command Injection (SLCI), where the attacker injects carefully crafted control commands into the swarm’s communication channels. By manipulating these commands, the adversary can simultaneously affect multiple drones, breaking down coordinated operations, inducing mission failures, or even causing physical damage.

**Swarm-Level Command Injection (SLCI)**

An example of such an offensive technique involves an attacker injecting spoofed control commands through the swarm’s communication protocol to redirect multiple drones to converge on a single GPS coordinate at the same altitude. This coordinated manipulation deliberately induces mid-air collisions, causing physical damage and effectively neutralizing the swarm’s operational capabilities. Alternatively, the attacker can force a mission abort by sending a command such as a “MISSION_CLEAR_ALL” packet, which irreversibly cancels the swarm’s active mission plan, leaving the drones idle or confused and preventing task completion.

By injecting critical MAVLink commands such as motor disarming, forced landing, mission clearing, or flight mode changes, an attacker controlling the swarm leader can effectively halt drone operations or abort the mission, thereby exerting full command and control over the swarm’s behavior.

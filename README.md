# Passive Telemetry Leakage in Unsecured ROS 2 Edge Systems

## 1. Experiment Title & The Core Analytical Question Asked
* **Experiment Title:** Passive Coordinate Data Leakage Auditing on Default DDS Deployments
* **Core Analytical Question:** By default, with zero security extensions configured on an open-source ROS 2 Jazzy installation, can an unprivileged network node passively capture clear-text spatial coordinate telemetry transmitted across a shared Wi-Fi network link without executing active Man-in-the-Middle (MITM) manipulations?

---

## 2. Executive Vulnerability Summary & Overall Risk Rating
* **Vulnerability Identifier:** Clear-text Data Transmission via Unencrypted DDS Protocol
* **Overall Risk Rating:** **HIGH**
* **Impact Analysis:** Successful exploitation yields total loss of data confidentiality. An adversary sniffing the network medium can reconstruct exact positional tracks (X, Y coordinates) of field robotics assets. This enables physical tracking, mission interception, and operational reconnaissance.
* **Technical Root Cause:** The default ROS 2 middleware implementation (eProsima Fast DDS) uses unencrypted Real-Time Publish-Subscribe (RTPS) UDP multicast/unicast structures. Data is transmitted via Common Data Representation (CDR) serialization without transport-layer or network-layer cryptographic protections.

---

## 3. Environmental Lab Diagram
```text
+---------------------------------------+             +-----------------------------------------+

|     TARGET DEVICE: FIELD ROBOT        |             |      AUDIT STATION: RESEARCHER          |
|  Hardware: Raspberry Pi 5 (16GB RAM)  |             |  Hardware: Host Laptop (Windows 11)     |
|  OS: Ubuntu Server 24.04.5 LTS        |             |  VM: Hyper-V Ubuntu 24.04 LTS Desktop   |
|  IP Address: 192.168.56.109           |             |  IP Address: 192.168.56.232             |
|  Software: ROS 2 Jazzy (Fast DDS)    |             |  Software: ROS 2 Jazzy, Wireshark, tcpdump|
+---------------------------------------+             +-----------------------------------------+

                    |                                                      |
                    | (UDP Multicast / Unicast)                            | (Passive Interception)
                    +-------------------[ Lab Wi-Fi Subnet ]---------------+
                                       (192.168.56.0/24)
```

---

## 4. Technical Step-by-Step Methodology
To guarantee complete laboratory reproducibility, execute the following commands in order across your respective terminal workspaces:

### Phase 1: Target Device Initialization (Window 1 - Pi 5 via SSH)
1. Navigate to the local exploits repository and ensure the ROS 2 environment workspace paths are correctly mapped:
   ```bash
   cd ~/dds-passive-capture/exploits
   source /opt/ros/jazzy/setup.bash
   ```
2. Launch the mock telemetry generation script to start broadcasting random field coordinate pairs every 3 seconds:
   ```bash
   python3 poc_payload.py
   ```

### Phase 2: Audit Station Verification (Window 2 - Ubuntu VM Desktop)
1. Verify network connectivity and automatic DDS discovery by launching the legitimate subscriber node:
   ```bash
   cd ~/dds-passive-capture/analysis
   source /opt/ros/jazzy/setup.bash
   python3 listener_node.py
   ```

### Phase 3: Traffic Interception Engine (Window 3 - VM Sniffer Terminal via Laptop SSH)
1. Map active link layer interfaces using `ip link` to verify the Hyper-V bridged interface (`eth0`).
2. Run `tcpdump` with a traffic filter to quietly record packets matching the target host IP while filtering out extraneous background noise:
   ```bash
   sudo tcpdump -i eth0 -w ~/dds-passive-capture/analysis/ros2_cleartext_coordinates.pcap udp and host 192.168.56.109
   ```
3. Maintain the monitoring window for 15 seconds to catch multiple packet rotations, then gracefully close the loop using `Ctrl + C`.

---

## 5. Observed Lab Results & Proof Images
Upon auditing the captured traffic file via Wireshark, the baseline hypothesis was verified. The ROS 2 traffic completely stripped variable naming attributes but dropped raw numbers on the wire inside predictable 24-byte packet submessages.

### Protocol Analysis Breakdown (Hex Dump View)
A structural frame audit of the `serializedData` payload within an isolated RTPS submessage highlighted the target leak sitting inside rows `0070` and `0080`:

```text
0070  00 00 00 01 00 00 [a7 e8 48 2e ff cd 58 40] [ca c3  ........H...X@..
0080  42 ad 69 1a 58 c0] 00 00 00 00 00 00 00 00 80 01  B.i.X...........
                        └─────── Z=0.0 ────────┘
```

### Forensic Data Reconstruction
By parsing the little-endian IEEE-754 double precision representation of the contiguous bytes, the following raw data points were extracted:
* **X-Coordinate Bytes:** `a7 e8 48 2e ff cd 58 40` $\rightarrow$ Evaluates mathematically to **`99.2177`**
* **Y-Coordinate Bytes:** `ca c3 42 ad 69 1a 58 c0` $\rightarrow$ Evaluates mathematically to **`-96.4127`**
* **Z-Coordinate Physical Anchor:** `00 00 00 00 00 00 00 00` $\rightarrow$ Evaluates mathematically to **`0.0`** (Validating structural alignment with `poc_payload.py`).

Refer to the visual proofs inside the repository folder:
* `media/capture-overview.png`: Demonstrates overall isolated `RTPS` data transactions captured inside the environment.
* `media/packet-details.png`: Provides a terminal screenshot of the raw hex dump mapping the leaky data offset.

---

## 6. Analytical Conclusion & Brief High-Level Remediation Strategy
### Analytical Conclusion
This lab demonstrates that default ROS 2 deployments lack structural transport security. Because security profiles must be explicitly enabled, standard installations expose mission-critical edge sensor telemetry directly to anyone on the local network link. 

### High-Level Remediation Strategy
To defend the network perimeter against passive sniffing vectors, engineers must transition from unencrypted DDS to **Secure ROS 2 (SROS2)** mechanisms. 
1. **Authentication:** Implement Public Key Infrastructure (PKI) by binding every endpoint node to a validated x.509 cryptographic certificate.
2. **Access Control:** Formulate detailed XML governance and permission structures to restrict unauthorized topic joining.
3. **Cryptographic Protection:** Enable native DDS Security extensions to enforce transport-layer **DTLS/AES-GCM encryption**, completely transforming clear-text binary parameters into opaque ciphertext.

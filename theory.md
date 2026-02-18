## 1. Introduction

The 5G New Radio (NR) system requires proper connectivity between the Radio Access Network (RAN) and the 5G Core Network (5GC). This experiment focuses on two fundamental procedures:

- **NG Setup Procedure**: Establishes the control plane connection between gNodeB (gNB) and Access and Mobility Management Function (AMF)
- **N3 Interface Activation**: Enables user plane data transmission between gNB and User Plane Function (UPF)

---

## 2. 5G System Architecture Overview

### 2.1 Network Components

**NG-RAN (Next Generation Radio Access Network):**
- gNodeB (gNB): Provides radio coverage and interfaces with 5G core

**5GC (5G Core Network):**
- AMF: Handles mobility and connection management
- UPF: Routes and forwards user data packets
- SMF: Manages PDU sessions

<img src="images/fig1.svg" alt="5G System Architecture" width="45%">

*Fig: 5G System Architecture - NG Setup and N3 Interface*

### 2.2 NG Interface Components

**NG-C (NG Control Plane):**
- Connects gNB to AMF
- Uses NGAP (NG Application Protocol)
- Carries control/signaling messages

**NG-U / N3 (NG User Plane):**
- Connects gNB to UPF
- Uses GTP-U (GPRS Tunneling Protocol - User Plane)
- Carries actual user data

<img src="images/fig2.svg" alt="NG Interface Components" width="45%">

*Fig: NG Interface Components*

---
<details>
<summary><strong>3. NG Setup Procedure</strong></summary>

## 3. NG Setup Procedure

### 3.1 Purpose

The NG Setup procedure is the first signaling exchange between gNB and AMF that establishes control plane connectivity.

- Exchange identities between gNB and AMF
- Share supported capabilities and network slices
- Verify service area compatibility
- Enable UE connection handling

### 3.2 NG Setup Message Flow

<img src="images/fig3.svg" alt="NG Setup Procedure Sequence" width="45%">

*Fig: NG Setup Procedure Sequence*

**Process Steps:**
- SCTP association established between gNB and AMF
- gNB sends NG SETUP REQUEST
- AMF validates the request
- AMF sends NG SETUP RESPONSE (or FAILURE)
- NGAP connection is active

### 3.3 NG Setup Request Message

**Key Information Elements:**

1. **Global RAN Node ID**
   - PLMN Identity: MCC + MNC (e.g., '00F110'H for MCC=001, MNC=01)
   - gNB ID: Unique identifier (e.g., '0012345'H)

2. **Supported TA List**
   - TAC (Tracking Area Code): e.g., '000064'H (TAC=100)
   - Broadcast PLMN List
   - TAI Slice Support List (S-NSSAI with SST)
     - SST (Slice/Service Type): e.g., '01'H for eMBB

3. **Default Paging DRX**
   - Values: v32, v64, v128, v256 (radio frames)
   - Example: v128 = check paging every 1.28 seconds

4. **RAN Node Name**
   - Human-readable identifier (e.g., "gnb0012345")

**Example:**

```
NG SETUP REQUEST
{
    Global RAN Node ID:
    {
        PLMN Identity: 00101
        gNB ID: 0012345
    }
    
    Supported TA List:
    {
        TAC: 000064
        S-NSSAI: { SST: 01 }
    }
    
    Default Paging DRX: v128
}
```

### 3.4 NG Setup Response Message

**Key Information Elements:**

1. **AMF Name**
   - FQDN format: "amarisoft.amf.5gc.mnc001.mcc001.3gppnetwork.org"

2. **Served GUAMI List**
   - GUAMI components:
     - PLMN Identity
     - AMF Region ID: e.g., '80'H (128 decimal)
     - AMF Set ID: e.g., 4
     - AMF Pointer: e.g., 1

3. **Relative AMF Capacity**
   - Value: 0-255 (higher = greater capacity)
   - Used for load balancing

4. **PLMN Support List**
   - Supported PLMNs and network slices
   - Must overlap with gNB's capabilities

**Example:**

```
NG SETUP RESPONSE
{
    AMF Name: "amarisoft.amf.5gc.mnc001.mcc001.3gppnetwork.org"
    
    Served GUAMI List:
    {
        PLMN Identity: 00101
        AMF Region ID: 128
        AMF Set ID: 4
        AMF Pointer: 1
    }
    
    Relative AMF Capacity: 50
    
    PLMN Support List:
    {
        PLMN Identity: 00F110
        S-NSSAI: { SST: 01 }
    }
}
```

### 3.5 NGAP Protocol Stack

<img src="images/fig4.svg" alt="NGAP Protocol Stack" width="45%">

*Figu: NGAP Protocol Stack*

**Layers:**
- **NGAP**: Application layer for NG-C signaling
- **SCTP**: Reliable transport (Port 38412)
- **IP**: Network layer routing
- **Ethernet**: Physical connectivity

---
</details>


<details>
<summary><strong>4. N3 Interface and ActivationNG Setup Procedure</strong></summary>
## 4. N3 Interface and Activation

### 4.1 N3 Interface Overview

The N3 interface is the user plane interface between gNB and UPF.

**Purpose:**
- Carries user data between UE and Data Networks
- Uses GTP-U tunneling protocol
- Operates independently of control plane

**Key Characteristics:**
- Tunnel-based using TEIDs (Tunnel Endpoint Identifiers)
- Bidirectional (uplink and downlink)
- Separate tunnel for each PDU session

<img src="images/fig5.svg" alt="N3 Interface Position in 5G Network" width="45%">

*Fig: N3 Interface Position in 5G Network*

### 4.2 N3 Activation Through PDU Session Setup

N3 activation occurs during PDU Session Resource Setup.

<img src="images/fig6.svg" alt="N3 Activation via PDU Session Setup" width="45%">

*Fig: N3 Activation via PDU Session Setup*

### 4.3 PDU Session Resource Setup Request

**Key Information Elements:**

1. **PDU Session ID**
   - Unique identifier (0-255), e.g., PDU Session ID = 1

2. **S-NSSAI**
   - Network slice identifier (SST: '01'H)

3. **UL NG-U UP TNL Information**
   - UPF IP Address: e.g., 10.0.0.162
   - Uplink TEID: e.g., 4f485cc3
   - gNB uses these to send uplink data to UPF

4. **PDU Session Type**
   - IPv4, IPv6, or Ethernet

5. **QoS Flow Setup Request List**
   - QFI (QoS Flow Identifier): e.g., 1
   - 5QI: e.g., 9 (default bearer)

**Example:**

```
PDU SESSION RESOURCE SETUP REQUEST
{
    PDU Session ID: 1
    S-NSSAI: { SST: 01 }
    
    UL NG-U UP TNL Information:       ← N3 UPLINK TUNNEL
    {
        Transport Layer Address: 10.0.0.162  (UPF IP)
        GTP-TEID: 4f485cc3                   (UL TEID)
    }
    
    PDU Session Type: IPv4
    
    QoS Flow: { QFI: 1, 5QI: 9 }
}
```

### 4.4 PDU Session Resource Setup Response

**Key Information Elements:**

1. **DL QoS Flow Per TNL Information**
   - gNB IP Address: e.g., 10.0.0.185
   - Downlink TEID: e.g., a968d0db
   - UPF uses these to send downlink data to gNB

2. **Associated QoS Flow List**
   - Successfully established flows (e.g., QFI=1)

**Example:**

```
PDU SESSION RESOURCE SETUP RESPONSE
{
    PDU Session ID: 1
    
    DL QoS Flow Per TNL Information:   ← N3 DOWNLINK TUNNEL
    {
        Transport Layer Address: 10.0.0.185  (gNB IP)
        GTP-TEID: a968d0db                   (DL TEID)
    }
    
    Associated QoS Flow List: { QFI: 1 }
}
```

**Result:** N3 tunnel is now fully configured for bidirectional data flow.

### 4.5 N3 Protocol Stack

<img src="images/fig7.svg" alt="N3 Interface Protocol Stack" width="45%">

*Fig: N3 Interface Protocol Stack*

**Layers:**
- **User Data**: User IP packets
- **GTP-U**: Tunneling protocol with TEID
- **UDP**: Port 2152
- **IP**: Source/Destination addressing
- **Ethernet**: Physical transport

### 4.6 N3 Data Flow

#### 4.6.1 Uplink Flow (UE → Data Network)

<img src="images/fig8.svg" alt="Uplink Data Flow Through N3" width="45%">

*Fig: Uplink Data Flow Through N3*

**Steps:**
1. UE sends IP packet (e.g., to 8.8.8.8)
2. gNB receives on radio interface
3. gNB encapsulates with GTP-U:
   - [Outer IP: gNB (10.0.0.185) → UPF (10.0.0.162)]
   - [UDP: Port 2152]
   - [GTP-U Header: UL TEID = 4f485cc3]
   - [Inner IP Packet: UE → 8.8.8.8]
4. Packet transmitted over N3 interface
5. UPF decapsulates (removes outer headers)
6. UPF forwards to Data Network

#### 4.6.2 Downlink Flow (Data Network → UE)

<img src="images/fig9.svg" alt="Downlink Data Flow Through N3" width="45%">

*Fig: Downlink Data Flow Through N3*

**Steps:**
1. Data Network sends response (e.g., from 8.8.8.8)
2. UPF receives packet
3. UPF encapsulates with GTP-U:
   - [Outer IP: UPF (10.0.0.162) → gNB (10.0.0.185)]
   - [UDP: Port 2152]
   - [GTP-U Header: DL TEID = a968d0db]
   - [Inner IP Packet: 8.8.8.8 → UE]
4. Packet transmitted over N3 interface
5. gNB decapsulates (removes outer headers)
6. gNB forwards to UE over radio

**Key Fields:**
- **Version**: 001 (GTP version 1)
- **Message Type**: 255 for user data (G-PDU)
- **TEID (32 bits)**: Most critical field
  - Uplink: UL TEID assigned by UPF
  - Downlink: DL TEID assigned by gNB
  - Used to identify PDU session

---
</details>

<details>
<summary><strong>5. Relationship Between NG Setup and N3</strong></summary>
## 5. Relationship Between NG Setup and N3

### 5.1 Dependency

<img src="images/fig10.svg" alt="NG Setup to N3 Activation Flow" width="45%">

*Fig: NG Setup to N3 Activation Flow*

**Key Points:**
- NG Setup is prerequisite: Must complete before N3 can be activated
- Different protocols: NG Setup uses NGAP, N3 uses GTP-U
- Different endpoints: NG Setup (gNB-AMF), N3 (gNB-UPF)
- One-to-many: One NG Setup supports multiple N3 tunnels

### 5.2 Complete Call Flow

1. **Network Setup Phase**
   - Physical connectivity established
   - SCTP association created
   - NG Setup Request/Response exchanged
   - NG-C interface ready

2. **UE Attachment Phase**
   - UE registration via NGAP
   - Authentication and security

3. **N3 Activation Phase**
   - PDU Session requested
   - Setup Request with UL tunnel info (UPF IP, UL TEID)
   - Setup Response with DL tunnel info (gNB IP, DL TEID)
   - N3 tunnels active

4. **Data Communication**
   - User data flows: UE ↔ gNB ↔ N3 ↔ UPF ↔ Data Network

---
</details>

<details>
<summary><strong>6. Summary</strong></summary>
## 6. Summary

### 6.1 NG Setup

**Purpose:** Establish control plane connectivity (gNB ↔ AMF)

**Key Messages:**
- NG Setup Request: Global RAN Node ID, Supported TAs, Paging DRX
- NG Setup Response: AMF Name, GUAMI List, PLMN Support List

**Protocol:** NGAP/SCTP/IP (Port 38412)

### 6.2 N3 Interface

**Purpose:** Carry user plane data (gNB ↔ UPF)

**Activation:** PDU Session Resource Setup procedure

**Key Parameters:**
- Request: UPF IP + Uplink TEID
- Response: gNB IP + Downlink TEID

**Protocol:** GTP-U/UDP/IP (Port 2152)

### 6.3 Importance

- **NG Setup**: Foundation for all signaling; enables UE connection
- **N3 Interface**: Carries actual user traffic; impacts user experience
- **Together**: Enable complete 5G connectivity (control + data)

---
</details>
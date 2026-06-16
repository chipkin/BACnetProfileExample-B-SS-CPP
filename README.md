---
title: "BACnet B-SS (Smart Sensor) Profile Example - C++"
description: "Minimal, copy-paste working example of a BACnet/IP Smart Sensor (B-SS) device in C++17 using the CAS BACnet Stack. Implements ReadProperty (DS-RP-B) and Who-Is/I-Am discovery."
keywords:
  - BACnet
  - BACnet/IP
  - BACnet device example
  - BACnet server example C++
  - how to implement a BACnet device
  - Smart Sensor
  - B-SS
  - device profile
  - ReadProperty
  - DS-RP-B
  - Who-Is
  - I-Am
  - CAS BACnet Stack
  - Chipkin
  - ANSI/ASHRAE 135
  - Protocol Revision 24
  - Network Port object
  - C++17
  - CMake
  - UDP 47808
language: C++
profile: B-SS (Smart Sensor)
services_enabled:
  - ReadProperty (DS-RP-B)
  - Who-Is / I-Am
platforms:
  - Windows
  - Linux
  - macOS
difficulty: Beginner
estimated_build_time: "A few minutes (first build compiles the stack)"
app_version: "1.0.0"
documented_for_stack_version: "5.4.2.0"
license: CC0-1.0
---

# BACnet B-SS (Smart Sensor) - C++ example

A minimal, copy-paste-friendly example showing how to implement the BACnet
**B-SS (BACnet Smart Sensor)** device profile in C++ using the
[CAS BACnet Stack](https://store.chipkin.com/services/stacks/bacnet-stack).
It listens on **BACnet/IP (UDP 47808)**, answers **ReadProperty** requests, and
is discoverable via **Who-Is / I-Am**.

> **Versions:** this document describes **example v1.0.0**, built and verified
> against **CAS BACnet Stack 5.4.2.0** at **Protocol_Revision 24**.

## What is a B-SS (BACnet Smart Sensor) profile?

A **device profile** is a standard "template" defined in Annex L of ANSI/ASHRAE
135. It lists the capabilities a class of device must support so that any
compliant client knows what to expect, and the BACnet Testing Laboratories (BTL)
certify devices against it. (New to BACnet in general? See Chipkin's
[What is BACnet?](https://docs.chipkin.com/protocols/bacnet/) guide.)

**B-SS (BACnet Smart Sensor)** is the simplest device profile - the standard
describes it as *"a simple sensing device with very limited resources."* It is
meant for inexpensive, fixed-function sensors (temperature, humidity, occupancy,
a contact, ...) that mostly just need to **report what they measure** when asked.

**What the profile requires:**

- **Data Sharing - ReadProperty - B side (DS-RP-B):** the device must answer
  **ReadProperty** requests for the values of its objects. This is the single
  mandatory BIBB (BACnet Interoperability Building Block) for B-SS.
- **Discovery:** the device must be findable, so it answers **Who-Is** with
  **I-Am**, and announces itself with an unsolicited I-Am at start-up.

**What the profile does NOT require** - and this example therefore omits on
purpose: **WriteProperty** (a smart sensor is read-only), **alarming / event
reporting**, **scheduling**, and **trending**.

**But it is still a full BACnet device.** Even the simplest profile must present
the standard object model - a **Device** object, a **Network Port** object (every
device needs one), and its sensor objects - and each object must expose all of
its **required properties**. The CAS BACnet Stack generates most of those
automatically (Object_Identifier, Object_Type, Status_Flags, Event_State,
Object_List, Protocol_*, ...); this example supplies the handful that are
application-specific. The result is conformant for **Protocol_Revision 24**.

## The device this example creates

```
Device 389001  "Rainbow"   (Vendor 389 - Chipkin Automation Systems)
    │
    ├── Analog Input 1       "Bronze"      Present_Value  21.5    (REAL, degrees Celsius)
    ├── Binary Input 1       "Emerald"     Present_Value  active  (0 = inactive / 1 = active)
    ├── Multi-State Input 1  "Hot Pink"    Present_Value  1       (state, 1..3)
    └── Network Port 1       "Vermilion"   the BACnet/IP port     (required on every device)
```

A BACnet client discovers `Rainbow`, then reads the present value and name of
each object:

```
Client  ──  Who-Is  ───────────────────────────▶   Rainbow (389001)
Client  ◀─  I-Am (389001, vendor 389)  ─────────    Rainbow
Client  ──  ReadProperty(AI 1, Present_Value)  ─▶   Rainbow
Client  ◀─  21.5  ──────────────────────────────    Rainbow
```

Object names follow this series' colour-naming convention (Device is always
"Rainbow").

## Prerequisites

You need a C++ toolchain, CMake, and this repository checked out **with
submodules** (the CAS BACnet Stack is a submodule).

### Windows

1. **C++ compiler** - install
   [Visual Studio Community](https://visualstudio.microsoft.com/downloads/)
   (free) and select the **"Desktop development with C++"** workload during
   setup. (Build Tools for Visual Studio also works.)
2. **CMake** - install from <https://cmake.org/download/> (Windows x64
   installer), or via `winget install Kitware.CMake`. Verify in a new
   PowerShell: `cmake --version` (need >= 3.15).
3. **The code with submodules**:
   ```powershell
   git clone --recursive https://github.com/chipkin/<this-repo>.git
   # already cloned without --recursive? run:
   git submodule update --init --recursive
   ```

### Linux / macOS

1. **C++ compiler & CMake**:
   - Debian/Ubuntu: `sudo apt install build-essential cmake git`
   - macOS: `xcode-select --install` and `brew install cmake`
   - Verify: `cmake --version` (need >= 3.15) and `c++ --version`.
2. **The code with submodules** - same `git clone --recursive` /
   `git submodule update --init --recursive` as above.

No prebuilt library or DLL is needed - this example compiles the stack from
source.

## Build

From this folder (`examples/BACnetProfileExample-B-SS-CPP/`):

```bash
cmake -B build -S .
cmake --build build --config Release
```

> **First build takes a few minutes** - it compiles the entire CAS BACnet Stack
> (~460 source files) once. Incremental rebuilds after that are fast.

**Building a standalone copy** (outside this repo)? Point CMake at your own copy
of the stack:

```bash
cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack
```

## Run

```bash
# Linux / macOS
./build/BACnetExampleBSS

# Windows
.\build\Release\BACnetExampleBSS.exe
```

Expected output:

```
BACnet B-SS (Smart Sensor) Example - C++ v1.0.0
CAS BACnet Stack version: 5.4.2.0
FYI: Listening for BACnet/IP on UDP port 47808.
FYI: Device 389001 ("Rainbow") ready. Vendor ID 389. Press 'h' for help.
TX 21 bytes to 192.168.3.255:47808 (broadcast)
```

The last line is the start-up I-Am the device broadcasts to announce itself. It
goes to the **local subnet broadcast** address (here `192.168.3.255`, computed
from the Network Port's interface), not the global `255.255.255.255`. As clients
talk to the device you'll see `RX ... bytes from ...` and `TX ... bytes to ...`
lines showing the traffic.

The device listens on UDP **47808** (BACnet/IP). Allow that port through your
firewall. To use a different port, pass `--port` (see below).

### Command-line options

| Option | Default | Meaning |
|--------|---------|---------|
| `--port <n>` | `47808` | UDP port to listen on (BACnet/IP). |
| `--deviceID <n>` | `389001` | The device's BACnet instance number (BACnet requires this to be configurable). |

### Interactive commands

While the example runs, these keys are available (shared across all examples in
the series):

| Key | Action |
|-----|--------|
| `h` | Show the version information and this command list. |
| `q` | Quit. |
| up arrow | Increase Analog Input 1 (`Bronze`) by 1.1. |
| down arrow | Decrease Analog Input 1 (`Bronze`) by 1.1. |

The up/down keys change the live `Present_Value` of the analog input, so a client
re-reading it sees the new value.

## Verify

Use a BACnet client such as the
[**CAS BACnet Explorer**](https://store.chipkin.com/products/tools/cas-bacnet-explorer):

1. **Discover** - send a **Who-Is**. The device replies with **I-Am** from
   instance **389001** (vendor **389**). It also broadcasts an I-Am at start-up.
2. **Browse the object model** - the device shows five objects: the Device
   (`Rainbow`), the three sensors, and the Network Port (`Vermilion`). Reading
   the Device's `Object_List` returns all five. The Network Port reports real
   BACnet/IP addressing - `IP_Address`, `IP_Subnet_Mask`, `BACnet_IP_UDP_Port`,
   and a `MAC_Address` the stack builds from them.
3. **Read the Device** - ReadProperty `389001` -> `Object_Name` returns
   `"Rainbow"`; `Protocol_Revision` returns `24`; `Description` returns the
   profile description string.
4. **Read a sensor** - ReadProperty Analog Input `1` -> `Present_Value` returns
   `21.5`; `Units` returns `degrees-Celsius`; `Out_Of_Service` returns `false`;
   `Object_Name` returns `"Bronze"`. Repeat for Binary Input `1` (`"Emerald"`,
   has `Polarity`) and Multi-State Input `1` (`"Hot Pink"`, has `Number_Of_States`
   = 3). Every required property of every object is readable.
5. **Confirm the profile boundary** - a **WriteProperty** to any object is
   rejected. That is correct: a B-SS Smart Sensor is read-only.

## Troubleshooting

| Symptom | Cause / fix |
|---------|-------------|
| CMake error: *"CAS BACnet Stack source not found"* | Submodules not initialized. Run `git submodule update --init --recursive` (or pass `-D CAS_STACK_DIR=...`). |
| `CASBACnetStackDLL.h: No such file or directory` | Same - submodules not checked out. |
| Windows: *"No CMAKE_CXX_COMPILER could be found"* | Install Visual Studio with the "Desktop development with C++" workload, then re-run from a fresh terminal. |
| First build seems stuck for minutes | Normal - it's compiling ~460 stack files. Only the first build is slow. |
| App prints *"Failed to bind UDP port 47808"* | Another BACnet program is already using 47808. Stop it, or run with `--port <n>`. |
| Client sends Who-Is but sees no I-Am | Firewall is blocking UDP 47808, or the client and device are on different subnets (Who-Is is a broadcast). Allow the port; test on the same subnet first. |
| Replies show an unexpected device instance or vendor | Another BACnet device is already running on this host/port (the socket uses `SO_REUSEADDR`, so several can share 47808). Stop the other device, or run this example on its own machine/IP. |

## Extending the example

The example is intentionally small so it's easy to change.

**Change a sensor's value or name** - edit the constants / callbacks in
`main.cpp` (e.g. the initial value of `g_analogInput1Value`, or the `"Bronze"`
string in `GetPropertyCharString`).

**Add a second analog input** - three small edits in `main.cpp`:

```cpp
// 1) a new instance number (in section 1)
static const uint32_t ANALOG_INPUT_2_INSTANCE = 2;   // "Silver"

// 2) add the object (in main, next to the other BACnetStack_AddObject calls)
BACnetStack_AddObject(DEVICE_INSTANCE, OBJECT_TYPE_ANALOG_INPUT, ANALOG_INPUT_2_INSTANCE);

// 3) serve its value + name (in the matching callbacks)
//    GetPropertyReal:        AI/2 + Present_Value -> *value = 23.1f;
//    GetPropertyCharString:  AI/2 + Object_Name   -> "Silver"
```

Rebuild, and the new sensor is readable. Going beyond reading (writable points,
outputs, COV, alarms) means implementing a richer profile.

## References

- **ANSI/ASHRAE Standard 135** (BACnet) - the protocol standard. Object model
  (Clause 12), services (Clause 15), BACnet/IP (Annex J), device profiles
  (Annex L). Purchase / preview via the [ASHRAE store](https://www.ashrae.org/technical-resources/standards-and-guidelines).
- **What is BACnet?** - Chipkin's introduction:
  <https://docs.chipkin.com/protocols/bacnet/>.
- **CAS BACnet Stack** - product page and documentation:
  <https://store.chipkin.com/services/stacks/bacnet-stack>.
- **CAS BACnet Explorer** - client for testing this device:
  <https://store.chipkin.com/products/tools/cas-bacnet-explorer>.
- **Shared helper used by this example** - [`../common/README.md`](../common/README.md).

## Copy this into your own project

This example uses the shared helper in [`../common`](../common) (one copy, used
by every example in the series). To take it as a standalone project, copy **this
folder and `../common`** together, then point CMake at your copies of the helper
and the CAS BACnet Stack:

```bash
cmake -B build -S . \
  -D EXAMPLES_COMMON_DIR=/path/to/common \
  -D CAS_STACK_DIR=/path/to/cas-bacnet-stack
```

The example source code is dedicated to the public domain under
[CC0-1.0](LICENSE) - use it for anything, no attribution required. The CAS
BACnet Stack is a separate, commercially licensed product and is not covered by
CC0. See also [CHANGELOG.md](CHANGELOG.md) and [AGENTS.md](AGENTS.md).

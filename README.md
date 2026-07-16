# BACnet B-SS (Smart Sensor) - C++ example

A minimal, copy-paste-friendly example showing how to implement the BACnet
**B-SS (BACnet Smart Sensor)** device profile in C++ using the
[CAS BACnet Stack](https://store.chipkin.com/services/stacks/bacnet-stack).
It listens on **BACnet/IP (UDP 47808)**, answers **ReadProperty** requests, and
is discoverable via **Who-Is / I-Am**.

> **Versions:** this document describes **example v1.1.0**, built and verified
> against **CAS BACnet Stack 6.0.0.0** (the `6.x` branch) at **Protocol_Revision 24**, with `common/` helper **v1.1.0**.

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

Object names follow this series' colour-naming convention (Device is always
"Rainbow").

## What this example supports

The example implements exactly the capabilities below - and nothing more, which
is the point of a profile example. These capabilities satisfy the **B-SS (BACnet
Smart Sensor)** profile; because they are also the baseline required by
**B-GENERAL**, this example satisfies the **B-GENERAL** profile as well.

### BIBBs (BACnet Interoperability Building Blocks)

| BIBB | Description | Supported |
|------|-------------|:---------:|
| DS-RP-B | Data Sharing - ReadProperty - B | ✅ |
| DM-DDB-B | Device Management - Dynamic Device Binding - B | ✅ |
| DM-DOB-B | Device Management - Dynamic Object Binding - B | ✅ |

### Services (executed / B-side)

| Service | Notes |
|---------|-------|
| ReadProperty | Responds to property reads (DS-RP-B). |
| Who-Is / I-Am | Answers Who-Is with I-Am, and broadcasts an I-Am on start-up (DM-DDB-B). |
| Who-Has / I-Have | Answers Who-Has with I-Have (DM-DOB-B). |

### Object types

| Object type | Instance | Name |
|-------------|:--------:|------|
| Device | 389001 | Rainbow |
| Analog Input | 1 | Bronze |
| Binary Input | 1 | Emerald |
| Multi-State Input | 1 | Hot Pink |
| Network Port | 1 | Vermilion |

## Requires the CAS BACnet Stack (licensed product)

This example **builds against the CAS BACnet Stack, which is a commercial Chipkin
product** - it is not free or open source, and there is no public/trial build.
The stack is referenced here as the **private** git submodule
`submodules/cas-bacnet-stack`; you can only fetch and build it once you have a CAS
BACnet Stack license and access to that repository.

**To get the CAS BACnet Stack (and access to build this example), contact
Chipkin:** <https://store.chipkin.com/services/stacks/bacnet-stack> or
sales@chipkin.com.

You can still read all of this example's source on GitHub to evaluate the
approach and the amount of code involved.

## What's in this repository

This is a **self-contained** project. It ships:

- `main.cpp` - the example device.
- `common/` - the shared helper (UDP, callbacks, CLI, keyboard) vendored in.
- `submodules/cas-bacnet-stack/` - the **CAS BACnet Stack as a git submodule**
  (private; requires a license - see above). Compiled from source; no prebuilt
  library or DLL is shipped.

## Footprint & performance

This example statically compiles the **entire** CAS BACnet Stack into one
executable (no external runtime/DLL). Release-build sizes of the whole
application (stack + example):

| Platform | Binary | Size |
|----------|--------|------|
| Windows x64 (MSVC, Release) | `BACnetExampleBSS.exe` | ~2.6 MB |
| Linux x64 (GCC, Release) | `BACnetExampleBSS` | ~6 MB unstripped (`strip` cuts it substantially) |

These are whole-application sizes. The stack's flash/RAM footprint on a
constrained MCU, CPU cost per `BACnetStack_Tick()`, ReadProperty latency, and
the maximum number of objects depend on your target and configuration. For
embedded-sizing and benchmark figures, contact Chipkin -
<https://store.chipkin.com/services/stacks/bacnet-stack>.

## Prerequisites

- A C++17 compiler (MSVC, GCC, or Clang).
- CMake >= 3.15.
- Git (to fetch the stack submodule).

### Windows

- **C++ compiler** - install
  [Visual Studio Community](https://visualstudio.microsoft.com/downloads/)
  (free) and select the **"Desktop development with C++"** workload.
- **CMake** - from <https://cmake.org/download/>, or `winget install Kitware.CMake`.

### Linux / macOS

- Debian/Ubuntu: `sudo apt install build-essential cmake git`
- macOS: `xcode-select --install` and `brew install cmake`

## Get the code

Clone this repository **and its submodule** (the CAS BACnet Stack):

```bash
git clone --recursive https://github.com/chipkin/BACnetProfileExample-B-SS-CPP.git
cd BACnetProfileExample-B-SS-CPP

# already cloned without --recursive? fetch the submodule:
git submodule update --init --recursive
```

## Build

```bash
cmake -B build -S .
cmake --build build --config Release
```

> **First build takes a few minutes** - it compiles the entire CAS BACnet Stack
> (~460 source files) once. Incremental rebuilds after that are fast.

If your CAS BACnet Stack lives somewhere other than the bundled submodule, point
CMake at it: `cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack`.

## Run

```bash
# Linux / macOS
./build/BACnetExampleBSS

# Windows
.\build\Release\BACnetExampleBSS.exe
```

Expected output:

```
BACnet B-SS (Smart Sensor) Example - C++ v1.1.0
CAS BACnet Stack version: 6.0.0.0
Common helper (common/) version: 1.1.0
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

**Add a second analog input.** Read this whole recipe before starting — the last
step is the one that is easy to miss and the one BTL will fail you for.

> **Why there are four edits, not three.** Most of the `GetProperty*` callbacks
> match on **both** object type *and* instance (`objectInstance ==
> ANALOG_INPUT_INSTANCE`). A new instance therefore falls through **every** such
> check and the property read errors. `GetPropertyBool` is the exception: it
> matches on type only, so `Out_Of_Service` works for a new instance for free.
> That inconsistency is why a partly-added object *looks* fine — `Present_Value`
> and `Out_Of_Service` answer, `Units` does not.

```cpp
// 1) a new instance number (in section 1).
//    Naming: a second object of a type is "<Colour> 2" - so Analog Input 2 is
//    "Bronze 2", NOT a new colour. Each object TYPE owns one colour series-wide.
static const uint32_t ANALOG_INPUT_2_INSTANCE = 2;   // "Bronze 2"
static float g_analogInput2Value = 23.1f;            // its live value

// 2) add the object (in main, next to the other BACnetStack_AddObject calls).
//    Check the return, like every other stack call in this file.
if (!BACnetStack_AddObject(g_deviceInstance, OBJECT_TYPE_ANALOG_INPUT, ANALOG_INPUT_2_INSTANCE)) {
    printf("Error: Failed to add Analog Input 2 (Bronze 2).\n");
    return 1;
}

// 3) serve its Present_Value + Object_Name:
//    GetPropertyReal:        AI/2 + Present_Value -> *value = g_analogInput2Value;
//    GetPropertyCharString:  AI/2 + Object_Name   -> "Bronze 2"

// 4) DO NOT SKIP: serve its Units, in GetPropertyEnumerated.
//    Units is a REQUIRED property of an Analog Input. The existing check reads
//    `objectInstance == ANALOG_INPUT_INSTANCE`, which is instance 1 - so without
//    this, reading Analog Input 2's Units returns an ERROR and the object is
//    NON-CONFORMANT. It will still appear in the Object_List and its
//    Present_Value will read back perfectly, so the device looks healthy right
//    up until BTL certification.
//    GetPropertyEnumerated:  AI/2 + Units -> *value = ENGINEERING_UNITS_DEGREES_CELSIUS;
```

Then re-run the Verify steps above **against Analog Input 2**, not just Analog
Input 1 — read every required property (`Present_Value`, `Object_Name`, `Units`,
`Status_Flags`, `Event_State`, `Out_Of_Service`), which is exactly what catches a
missed step 4.

Going beyond reading (writable points, outputs, COV, alarms) means implementing a
richer profile — see B-SA and B-ASC.

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
- **Shared helper used by this example** - [`common/README.md`](common/README.md).

## Use this in your own project

This repository is self-contained: clone it (with the submodule) and build, then
copy what you need into your product. The example source code is dedicated to the
public domain under [CC0-1.0](LICENSE) - use it for anything, no attribution
required. The CAS BACnet Stack is a separate, commercially licensed product and
is not covered by CC0.

See also [CHANGELOG.md](CHANGELOG.md) and [AGENTS.md](AGENTS.md).

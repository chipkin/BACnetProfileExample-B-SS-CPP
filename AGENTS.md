# AGENTS.md

Guidance for AI coding agents working in this repository. See
<https://agents.md/> for the format. Human contributors should read
[README.md](README.md) first.

## What this project is

A **tutorial** C++ example that implements the BACnet **B-SS (Smart Sensor)**
device profile using the CAS BACnet Stack. It is one of a series - one git repo
per BACnet profile. The top priority is that the code reads like a tutorial a
customer can learn from and copy-paste. Favour clarity over cleverness.

## Dependencies (not in this repo)

This repo intentionally contains only the example's own files. Two things it
needs live elsewhere:

- **The CAS BACnet Stack** (compiled from source). Point CMake at it with
  `-D CAS_STACK_DIR=/path/to/cas-bacnet-stack`.
- **The shared example helper** (`common/`), normally at `../common` in the
  example collection. Point CMake at it with
  `-D EXAMPLES_COMMON_DIR=/path/to/common`.

## Build

```bash
cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack -D EXAMPLES_COMMON_DIR=/path/to/common
cmake --build build --config Release
```

The first build compiles the whole stack (~460 files) and takes a few minutes;
later incremental builds are fast.

## Run

```bash
./build/BACnetExampleBSS [--port 47808] [--deviceID 389001]   # Linux/macOS
.\build\Release\BACnetExampleBSS.exe [--port 47808] [--deviceID 389001]   # Windows
```

Interactive keys while running: `h` help, `q` quit, up/down nudge Analog Input 1.

## Conventions

- Device is named "Rainbow"; objects use the series' colour names; vendor id 389.
- Implement **only** the services and objects the B-SS profile requires - but
  expose **every required property** of each object for Protocol_Revision 24.
- Match the surrounding code style: `const`-correct parameters, check every stack
  return value, keep `main.cpp` linear and well-commented.
- Do **not** edit `../common` from this repo - it is shared across all examples
  and has a single source of truth.

## How to verify a change

There are no unit tests; verification is behavioural:

1. Build, then run one instance on a clear UDP port.
2. With a BACnet client (e.g. the CAS BACnet Explorer), send **Who-Is** and
   confirm **I-Am** from the device instance.
3. **ReadProperty** every required property of every object and confirm the
   values; confirm `Protocol_Revision` is 24 and `Object_List` lists all objects.
4. Confirm services that are not enabled (e.g. WriteProperty) are rejected.

## Releasing

Bump `APP_VERSION` in `main.cpp` and add an entry to [CHANGELOG.md](CHANGELOG.md),
then tag `vX.Y.Z`. The GitHub Actions workflow builds and publishes the release.

## License

The example source code is dedicated to the public domain under
[CC0-1.0](LICENSE). The CAS BACnet Stack is a separate, commercially licensed
product and is not covered by that dedication.

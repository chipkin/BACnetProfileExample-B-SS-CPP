# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-07-14

### Changed

- **CAS BACnet Stack pinned to the head of the `6.x` branch** (`14676437`).
  The previous pin was on a pre-6.x lineage; this brings ~248 commits of stack
  fixes and features. All examples in the series pin the same stack commit.
- `common/` updated to **v1.1.0**: the vendored helper now carries its own
  version (`COMMON_VERSION`, printed at start-up) and its own changelog
  (`common/CHANGELOG.md`), and `CASBACnetStackExampleConstants.h` is now the
  series-wide superset (identical file in every example).

### Fixed

- `APP_VERSION` had drifted: `main.cpp` still said `1.0.1` while the repo was
  tagged `v1.0.2`. From this release the two are kept in lock-step.

## [1.0.2] - 2026-06-16

### Fixed

- `GetPrimaryIPv4` (Windows): check the subnet-mask `inet_pton` return and
  zero-initialize the `in_addr`, so a mask string that fails to parse can no
  longer be read uninitialized (it falls back to the limited broadcast).

## [1.0.1] - 2026-06-16

### Fixed

- Stop manually enabling **required** object properties - the stack already
  enables them when the object is added (`AddObject` / `AddNetworkPortObject`).
  Only optional properties need `SetPropertyEnabled` now. (Resolves
  [#1](https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/issues/1).)
- Use `inet_pton` instead of the deprecated `inet_addr` (MSVC C4996).

### Added

- `State_Text` (optional) on Multi-State Input 1: "On", "Off", "Auto".
- README: a "Footprint & performance" section; a clear note that the CAS BACnet
  Stack is a **licensed product** (no public/trial build - contact Chipkin).
- Source comments: how to change the Vendor Identifier; how to feed live sensor
  readings into the property callbacks without blocking the tick loop.

### Changed

- Build warnings (`-Wall -Wextra` / `/W4`) on the example's own sources only.
- CI: a smoke-test step (starts, binds, stays up) before packaging.

## [1.0.0] - 2026-06-16

### Added

- Initial **B-SS (BACnet Smart Sensor)** profile example for the CAS BACnet Stack
  in C++.
- **Device** object "Rainbow" - default instance `389001`, vendor `389` (Chipkin
  Automation Systems) - with its full identity (name, description, vendor, model,
  firmware, application software version).
- Read-only sensor objects: **Analog Input 1** "Bronze" (REAL, degrees Celsius,
  starts at 21.5), **Binary Input 1** "Emerald" (active/inactive), **Multi-State
  Input 1** "Hot Pink" (state 1..3).
- **Network Port 1** "Vermilion" with full BACnet/IP addressing - `IP_Address`,
  `IP_Subnet_Mask`, `IP_Default_Gateway`, `BACnet_IP_UDP_Port`, `BACnet_IP_Mode`;
  `MAC_Address` is computed by the stack from the IP address and UDP port.
- **DS-RP-B** (ReadProperty) and automatic **Who-Is / I-Am** discovery; an
  unsolicited I-Am is broadcast to the local subnet on start-up.
- All **required properties for Protocol_Revision 24** across every object.
- Interactive keys: `h` help, `q` quit, up/down nudge Analog Input 1 by +/-1.1.
- Command-line options: `--port <n>` and `--deviceID <n>`.
- Cross-platform CMake build that compiles the CAS BACnet Stack from source.
- Self-contained repository: the shared helper is vendored in `common/`, and the
  CAS BACnet Stack is included as a git submodule at
  `submodules/cas-bacnet-stack` - clone with `--recursive` and build.
- GitHub Actions workflow that builds Windows + Linux and publishes a release on
  a `vX.Y.Z` tag.

[Unreleased]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/compare/v1.0.2...HEAD
[1.0.2]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/releases/tag/v1.0.0

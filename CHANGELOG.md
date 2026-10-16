# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - unreleased

Profile-functional (series bar): the sensor now performs the B-SS Profile
Function - ANSI/ASHRAE 135-2024 Annex L.4.5, "Ability to provide values for any
of its BACnet objects upon request" - with readings that change, and shows a
Failure Mode. The BACnet Surface is unchanged (DS-RP-B, DM-DDB-B, DM-DOB-B; the
same five objects).

### Added

- **Sensor Simulation** (`main.cpp` section 2, `SensorSimulationTick()`, labelled
  Simulation in the README): once a second Analog Input 1 moves 0.5 C between
  18.0 and 26.0, Binary Input 1 toggles every 3 s, Multi-State Input 1 steps
  1 -> 2 -> 3 -> 1 every 4 s. Before, Binary Input 1 and Multi-State Input 1
  were constants and Analog Input 1 changed only on an arrow key.
- **Analog Input 1 `Reliability`** (optional property, now enabled). It reads
  `no-fault-detected`, or `open-loop` while the simulated sensor wire is broken;
  the stack derives the FAULT bit of `Status_Flags` from it (cl. 12.2.7, 12.2.9).
  While broken the reading freezes. The wire is broken only through a test-only
  hook: the environment variable `CAS_EXAMPLE_TEST_SENSOR_FAULT_FILE` names a
  file, and the wire is broken while that file exists.
- **Wire Tests** `tests/sensing_test.py` (the readings follow the Simulation over
  12 s) and `tests/failure_modes_test.py` (broken and repaired wire; Error
  UNKNOWN_OBJECT / UNKNOWN_PROPERTY / INVALID_ARRAY_INDEX per cl. 15.5.1.3.1;
  WriteProperty rejected UNRECOGNIZED_SERVICE per cl. 18.9), with
  `tests/bss_setup.py`. Every check is mutation-verified (tests/README.md).

### Changed

- The up/down keys nudge the simulated Analog Input 1 reading; the Simulation
  carries on from the new value.
- README: the scenario, the Simulation, the Wire Test table; TUTORIAL: replacing
  the Simulation with hardware and reporting a sensor fault; PICS: Reliability
  row, the objects' notes, Application_Software_Version 1.3.0.

## [1.2.3] - unreleased

### Fixed

- **Network Port 1 properties a client could not read** (#20, #21, #22). Chipkin
  BACnet Explorer 4.23.2.0's discovery reads every property in each object's
  `Property_List`; three on Network Port 1 failed: `IP_DNS_Server` (406) with
  Abort `other`, `FD_BBMD_Address` (418) with Error `property` /
  `value-not-initialized`, `FD_Subscription_Lifetime` (419) with Error `object` /
  `read-access-denied`. The stack lists them by default; the fixes are in the
  stack (cas-bacnet-stack #2340 / PR #3436, #3387 / PR #3413, #2452), so the
  Stack Pin moves from `6.x` @ `abd4cee1` (6.0.21) to `6.x` @ `64b2cfac5`
  (6.0.23). No `main.cpp` change was needed. All three now read as values
  (`IP_DNS_Server` 0.0.0.0, `FD_BBMD_Address` a Null host with port 0,
  `FD_Subscription_Lifetime` 0).

### Added

- **Wire Tests** (`tests/`, README "Wire Tests"): `tests/discovery_test.py` does
  what a client's discovery does - Who-Is, the Device's `Object_List`, then every
  object's `Property_List` and a ReadProperty of every listed property - and fails
  on any Error, Reject, Abort or silence, naming 406/418/419 on Network Port 1
  separately. `tests/run_wire_tests.py` runs it in one command; CI runs it on
  Linux after the smoke test. Verified to fail at the old pin on exactly those
  three properties (7/11) and pass at the new one (11/11), on Windows and Linux.
- `wiretest/` 0.5.0, the series' shared Wiretest Harness, vendored verbatim.

### Changed

- **Protocol_Revision 24 -> 30** (the stack's default at the new pin).
- **Segmentation is now supported** (`segmented-both`, `Max_Segments_Accepted`
  16, `APDU_Segment_Timeout` 5000 ms): the stack's customer build compiles
  segmentation in from this pin (stack #2992). `docs/PICS.md` section 5 updated.
- Network Port 1's `Property_List` also lists `Command` (417) and
  `Command_Validation_Result` (510) at this pin, from the stack.
- README, AGENTS.md and PICS version lines: stack 6.0.23, `common/` 3.3.0,
  Protocol_Revision 30. APP_VERSION 1.2.2 -> 1.2.3.
- **`common/` 3.3.0 -> 3.4.0**, cherry-picked from B-SS-CPP `5d78299` (branch `common-3.4.0-broadcast-to-self`):
  `HelperSendMessage` now sends a broadcast the stack addresses to this device's
  own socket to the subnet broadcast, instead of unicasting it back to itself
  (stack #3933). No change in this example's Wire Test results.
- **`wiretest/` 0.5.0 -> 0.6.0**, copied verbatim from B-LS-CPP. Backward
  compatible: it adds codec helpers and the `peer`, `alarm`, `workstation` and
  `wait` modules this example does not use yet.
- **The discovery Wire Test retries its Who-Is once**, printing a visible
  `[RETRY]` line, when no I-Am comes back in 3 s. Who-Is is unconfirmed, so
  nothing else retries it, and the example's UDP receive can drop a datagram on
  Windows.

## [1.2.2] - unreleased

### Changed

- **Device renamed from the series' colour placeholder "Rainbow" to "Chipkin
  Example B-SS"** so devices from different examples in the series are
  distinguishable from each other on the same BACnet network - every example
  previously announced the identical Object_Name "Rainbow", which made two
  examples on one subnet indistinguishable by name. Sub-object names (Analog
  Input 1 "Bronze", etc.) are unchanged - only the Device object's name
  changed. `docs/colour-table.md` (series root) updated to match. APP_VERSION
  bumped 1.2.1 -> 1.2.2.

## [1.2.1] - unreleased

### Fixed

- **`Application_Software_Version` (12) and `Firmware_Revision` (44) were
  hardcoded and stale.** `Application_Software_Version` now reads
  `APP_VERSION` directly (one source of truth, can't drift from
  `--version`'s own banner again). `Firmware_Revision` is now built at
  runtime from the CAS BACnet Stack's own `BACnetStack_GetAPIMajorVersion()`/
  `GetAPIMinorVersion()`/`GetAPIPatchVersion()`/`GetAPIBuildVersion()` (the
  same 4 calls `common/CASExampleHelper.cpp`'s `PrintVersion()` already uses
  for the startup banner), populated once right after `LoadBACnetFunctions()`
  succeeds. Same fix as `BACnetProfileExample-B-SCHUB-CPP` v1.1.13, applied
  series-wide after a real device read via BACnet Explorer found the stale
  values.

## [Unreleased]

### Changed — documentation restructure, single-command build

- **README.md cut down to what this example is.** It now covers only the B-SS
  profile, the device this example creates, the BIBBs/services/objects it
  supports, how to build and run it, and how to verify it. It links to the
  releases page for prebuilt binaries near the top. Removed: the general
  explanation of what a device profile is, the "start here / part of the series"
  framing, the "Before you ship" table, the "Get the code" and "Link mode"
  sections, and the CC0 dedication paragraph.
- **New [TUTORIAL.md](TUTORIAL.md)** holds the long-form material the README used
  to carry: extending the example (including the silent-failure warning for
  adding a second object), what each object type needs the application to serve,
  who serves what, how to review a change against the conformance statement, and
  troubleshooting.
- **New [docs/PICS.md](docs/PICS.md)**, a Protocol Implementation Conformance
  Statement in the Annex A shape (product description, profile claimed, BIBBs,
  services, segmentation, object types, data link layer, address binding,
  networking, character sets), with the generated objects-and-properties tables
  as its final section. The README's `## Objects and properties` section is gone;
  it links here instead.
- **The Device object is now in `docs/objects.json`** and therefore in the
  generated tables, which previously covered only the four non-Device objects.
- **Build is `cmake -B build -S .` and nothing else, on every platform.** The
  documented build no longer calls `tools/build-stack-static.sh`, which lives in
  the example-series repository and is therefore not available to a customer who
  downloads this repository on its own. The example builds in the adapter's
  default SOURCE mode: the stack's sources are compiled into the executable, so
  there is no library or DLL to build first. `CMakeLists.txt`, `AGENTS.md` and
  the release workflow were updated to match, and the workflow now asserts
  `CAS_BACNET_STACK_LINK=SOURCE` rather than `STATIC`.
- **The "Before you ship" guidance moved into `main.cpp`**, next to the constants
  it applies to, including the warning that `Object_Name` is a compile-time
  constant and must be made per-unit configurable in a real product.
- **The series profile table is now a per-language link table** (one row per
  profile, one column per language) grouped by profile family, instead of a table
  of required BIBBs.

### Fixed

- `tools/gen-objects-properties.py` dropped the required `Property_List` row from
  every generated table: the filter that skips the reference table's `| Property |`
  header row also matched the `| Property_List |` data row.

## [1.2.0] - 2026-09-15

### Changed — STATIC link, generated README blocks

- **Stack re-pinned to `6.x` @ `abd4cee1` (reports itself as 6.0.21), tracking the
  `6.x` branch** (`.gitmodules` `branch = 6.x`), up from the squashed
  `issues/runbook` @ `670963ab` this example built against briefly. Same interface
  as `670963ab` (`abd4cee1` is the head of `6.x` after that squash) - no `main.cpp`
  change beyond the pin.
- **Links the CAS BACnet Stack as a prebuilt STATIC library**
  (`CAS_BACNET_STACK_LINK=STATIC`, built by `tools/build-stack-static.sh` from the
  stack's own project files) instead of compiling the stack from `source/` into
  this project. `CMakeLists.txt`, the README "Link mode" section and
  `AGENTS.md` now describe STATIC only; the adapter's SOURCE mode gets one
  line noting it exists. No DLL mode is documented or shipped.
- **`.github/workflows/release.yml` rewritten**: builds the STATIC library (cached
  on the submodule SHA), asserts `CAS_BACNET_STACK_LINK=STATIC` from
  `CMakeCache.txt`, and publishes `metrics-windows.json` / `metrics-linux.json`
  (binary size, SHA-256 prefix, start-up time to `ready`, stack commit, link
  mode, compiler) as release assets alongside the binaries.
- **README gained three generated/filled sections**: `## Objects and properties`
  (from `docs/objects.json` via `tools/gen-objects-properties.py`), `## The
  BACnet profile example series` (the series profile table, via
  `tools/sync-profile-table.sh`), and `## Footprint` (filled from
  `metrics-*.json` at release; currently the placeholder row).
- Fixed README staleness: the version call-out, expected-output block, and
  "What's in this repository" description had drifted to v1.1.0 / stack 6.0.0.0
  / common 1.5.1 and "no prebuilt library" while the code and `common/` had
  already moved to v1.2.0 / common 2.0.0; all now agree with what the binary
  prints.

### Changed — updated to the current CAS BACnet Stack interface

Four interface changes reach this example versus the prior `6.x-TestTool` pin
`756371c1`; the full list, with before/after signatures, is on cas-bacnet-stack
issue #1641.

- **Every `GetProperty*` callback gained a trailing `uint32_t* errorCode`**
  (stack issue #974). The stack presets it to `success` and reads it only on a
  `false` return, so a declining callback can now name the BACnet error the
  client receives. `main.cpp` uses it in exactly one place — `State_Text` with an
  out-of-range array index now answers `Error(property, invalid-array-index)`
  instead of an empty string — and deliberately leaves it alone on every
  catch-all `return false`, because the stack's decline-and-fabricate fallback is
  what answers required properties this application does not serve (the Device's
  `Max_APDU_Length_Accepted`, `APDU_Timeout` and `Number_Of_APDU_Retries`). The
  commentary above the callbacks explains the trade-off.
- **`BACnetStack_AddNetworkPortObjectWithNetworkNumber()` is gone**, folded into
  `BACnetStack_AddNetworkPortObject()`, which now always takes the network number
  and its quality. Same arguments, one function.
- **Links are identified by Network Port object instance, not network type**
  (stack issues #822/#556) — the transport callbacks and `SendIAm` all changed.
  Handled in `common/`; `main.cpp` calls the new
  `CASExampleHelper::SetNetworkPortInstance()`.
- **`GetSystemTime` returns `CASBACnetTime` (`int64_t`) instead of `time_t`.**
  Handled in `common/`.

`common/` goes to **2.0.0** (breaking; see `common/CHANGELOG.md`) and must be
re-copied into every example in the series.

### Fixed (upstream)

- Building this example against the stack surfaced a compile break in
  `BACnetInterface.cpp`: the per-port rename left eight `NOREF(networkType)`
  calls naming a parameter that no longer exists, in code that only compiles
  when the matching `STACK_OPTION_*` is off. `SendWriteGroup`'s is the one this
  build compiles, so the example could not be built at all. Fixed upstream in
  cas-bacnet-stack PR #1759, which is included in the pin above.

### Verified

Built on Windows/MSVC and exercised against a BACnet client: Who-Is → I-Am
(device 389001, APDU 1476, vendor 389); ReadProperty of every required property
of all five objects returns the expected value, `Protocol_Revision` is 24 and
`Object_List` lists all five; `State_Text[1..3]` reads `On`/`Off`/`Auto` and
`State_Text[4]` errors with `invalid-array-index`; WriteProperty and the other
non-B-SS services are answered `Reject(unrecognized-service)`.

## [1.1.0] - unreleased

> Not tagged yet: the newest tag here is `v1.0.2`. `release.yml` publishes binaries on a `v*.*.*`
> tag, so until that tag exists this section describes what is on the
> branch, not what shipped.

### Changed

- **Links the CAS BACnet Stack through the `CASBACnetStack::Adapter` CMake target
  instead of compiling its `source/*.cpp` into this project directly.** `main.cpp`
  and `common/CASExampleHelper.cpp` now include `CASBACnetStackAdapter.h` and call
  `LoadBACnetFunctions()` once at the top of `main()`; **every `BACnetStack_*` call
  site is unchanged** — the adapter exposes the same export names in every link
  mode. `CAS_BACNET_STACK_LINK` (`SOURCE` default, or `STATIC`/`DLL`) now picks the
  link mode, so switching is a CMake flag rather than a code change. See the
  README's new "Link modes" section.
  - Stack pinned to `6.x-TestTool` @ `756371c1`, which carries the adapter
    (cas-bacnet-stack PRs #267 and #268).
  - `common/` bumped to **v1.5.1** (see `common/CHANGELOG.md`). The
    `LoadBACnetFunctions()` requirement is a contract change shared by every
    example in the series.
  - Release CI now passes `-DCAS_BACNET_STACK_LINK=SOURCE` **explicitly** and
    asserts it back out of `CMakeCache.txt`, so a published artifact stays a
    single self-contained executable even if the CMake default ever moves.
- **Documentation corrections** carried over from the B-ASC review: the version
  banner and sample output now match the shipped `common/` version (they claimed
  v1.3.0); the Troubleshooting table quoted a CMake error string that no longer
  exists and blamed `SO_REUSEADDR` for a symptom that on Windows surfaces as a bind
  failure (the socket asks for `SO_EXCLUSIVEADDRUSE`); added parallel-build
  guidance for the ~600-file first compile.

### Changed

- **CAS BACnet Stack pinned to the head of the `6.x` branch** (`14676437`).
  The previous pin was on a pre-6.x lineage; this brings ~248 commits of stack
  fixes and features. (The series is mid-migration to 6.x, so a few examples
  still pin the 5.x line; see the runbook's pin table for the current split.)
- `common/` is vendored at **v1.3.0** (see `common/CHANGELOG.md`): it carries its own
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

[1.0.2]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/chipkin/BACnetProfileExample-B-SS-CPP/releases/tag/v1.0.0

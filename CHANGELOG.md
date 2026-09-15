# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - unreleased

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

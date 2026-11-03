# TODO

Open items and known limits at 1.4.1. Nothing here stops the example from
building, running or answering a client as [README.md](README.md) describes. Where
a line has a `ref:` tag, quote it when you contact Chipkin support.

## Known gaps

- **`common/` sends a frame the stack marks as a broadcast to the one address the stack
  gave, as a unicast.** The shared `common/`
  helper (the same copy is in every example of the series) does this, so a router or
  gateway's reply that cl. 6.4.2 requires to be a broadcast goes only to the
  requester. No effect on this sensor has been observed. Fix: change `common/`, then
  copy it into every example.
- **The device hears its own broadcasts.** The operating system delivers a datagram
  sent to the subnet broadcast address back to the sending socket, so the start-up
  I-Am (and any Who-Is answer) is received by the device itself. The stack logs this
  as `Error:` lines at start-up (see the README); it is harmless on a sensor. Since
  stack 6.x `1e7dffe2b` a router's BACnet/IP port drops its own looped-back
  broadcast, so on a router this no longer becomes a broadcast storm; a device with
  no router port, like this one, still receives its own. (ref:
  chipkin/cas-bacnet-stack#4604)

## Not implemented, on purpose

- **No WriteProperty, no COV, no alarming, no scheduling, no trending.** A B-SS Smart
  Sensor is read-only (ANSI/ASHRAE 135-2024 Annex L.4.5), and a WriteProperty gets a
  Reject `unrecognized-service`.
- **The Device's `Object_Name` and instance are set at start-up** (`--deviceName`,
  `--deviceID`), not over the network, and are not kept across a restart. Start the
  example with the same options each time.
- **The sensor readings are simulated.** Replace `SensorSimulationTick()` with your
  own sampling to use real hardware (TUTORIAL.md explains how).

## VeriBACnet

- A VeriBACnet run of this project (43 BTL tests) is 34 PASS, 0 FAIL, 9 SKIP; see
  `tests/veribacnet/RESULTS.md`. Four of the skips need an operator to restart the
  device with another name or instance, and five do not apply to a device that cannot
  create or delete objects.

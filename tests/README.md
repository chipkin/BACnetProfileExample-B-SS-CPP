# B-SS Wire Tests

Tests that drive the running smart sensor over BACnet/IP from outside it. A
B-SS device's Profile Function is to report what it measures to any client that
asks, so the test asks for everything, the way a client discovering the device
does. They are written against the series' shared harness in
[`../wiretest/`](../wiretest/README.md) (vendored, like `common/`); everything
specific to this sensor is here. Python 3 standard library only - nothing to
`pip install`.

## Running

Build the example (README "Build"), then from the repository root:

```bash
python3 tests/run_wire_tests.py --exe build/BACnetExampleBSS
```

On Windows, `--exe build\Release\BACnetExampleBSS.exe`. It ends with
`ALL WIRE TESTS PASSED` (exit 0) or the scripts that failed. `--port` (default
47821) is the UDP port the test starts the sensor on. It takes a few seconds.
CI (`.github/workflows/release.yml`) runs it on Linux after the smoke test.

## The setup

```
test client (127.0.0.1, ephemeral UDP port)  - the Peer
    |  BACnet/IP, unicast
BACnetExampleBSS  UDP --port                 - Device 389001
```

The Peer is a Python BACnet client (`wiretest.Client`), never another example's
binary. It plays a BACnet client such as Chipkin BACnet Explorer.

## `discovery_test.py` - every listed property reads as a value

1. **Who-Is** (for 389001, unicast) -> the device's I-Am.
2. **Device `Object_List`** names exactly the Device, Analog Input 1, Binary
   Input 1, Multi-state Input 1 and Network Port 1. This is the control: a walk
   over nothing would pass.
3. For each object, **`Property_List`** reads, and a ReadProperty of every
   property it lists - plus `Object_Identifier`, `Object_Name`, `Object_Type`
   and `Property_List`, which cl. 12.1.1.4.1 leaves out of the list - returns a
   ComplexACK. An Error, Reject, Abort or no answer prints an `[UNREADABLE]` line
   and fails the check. 135-2024 cl. 12.1.1.4.1: Property_List holds "one
   property identifier for each property that exists within the object".
4. Network Port 1's **`IP_DNS_Server` (406)**, **`FD_BBMD_Address` (418)** and
   **`FD_Subscription_Lifetime` (419)** are each listed and read as a value - one
   check per issue (#20, #21, #22), the three reads Explorer 4.23.2.0 found
   failing.

## It fails without the fix

The fix for #20-#22 is in the stack, so "without the fix" is the old Stack Pin.
Run 2026-10-15, the same `main.cpp`, stack `6.x` @ `abd4cee1` (6.0.21) against
`6.x` @ `64b2cfac5` (6.0.23):

| Stack | Windows (STATIC build) | Linux (WSL Ubuntu 22.04, SOURCE build) |
|---|---|---|
| `abd4cee1` | 7/11 - FAIL | 7/11 - FAIL |
| `64b2cfac5` | 11/11 - PASS | 11/11 - PASS |

At `abd4cee1`, on both platforms, the failures were exactly these three reads:

```
[UNREADABLE] Network Port 1 property 406: Abort reason=0
[UNREADABLE] Network Port 1 property 418: Error class=2 code=72
[UNREADABLE] Network Port 1 property 419: Error class=1 code=27
[FAIL] every property of every object reads as a value (74 reads) - 3 unreadable: Network Port 1 406, Network Port 1 418, Network Port 1 419
[FAIL] Network Port 1 IP_DNS_Server (406): listed in Property_List and reads as a value (issue #20) - Abort reason=0
[FAIL] Network Port 1 FD_BBMD_Address (418): listed in Property_List and reads as a value (issue #21) - Error class=2 code=72
[FAIL] Network Port 1 FD_Subscription_Lifetime (419): listed in Property_List and reads as a value (issue #22) - Error class=1 code=27
```

Abort reason 0 is `other`; Error class 2 code 72 is `property` /
`value-not-initialized`; class 1 code 27 is `object` / `read-access-denied` -
what Explorer reported. At `64b2cfac5` the walk is 78 reads, all values.

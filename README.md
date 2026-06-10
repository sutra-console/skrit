# skrit

The **contract** the [Sutra](https://github.com/sutra-console/sutra) app and
[Duta](https://github.com/sutra-console/duta) firmware both speak. A *skrit*
device is **any** device that speaks this wire protocol and answers the
self-describe commands — regardless of MCU, framework, or transport.

> *skrit* — the tail of **San·skrit** (the naming family), read as **script**,
> echoing **writ**: the written agreement two parties share.

- **[PROTOCOL.md](PROTOCOL.md)** — the full spec: COBS/CRC framing, message
  types, self-describe, typed DATA streams (`DATA_DESC`), serial control, PWM
  config, runtime IO provisioning (`PIN_CAPS` / `CONFIG_*`), reboot, skrit-mux,
  network auth, async events, the skrit-mc macro bytecode, and macro storage.
- **[protocol.h](protocol.h)** — portable C reference (message IDs, status /
  capability / flag / pin-capability bits, CRC-8/ATM, COBS). Used by the C/C++
  firmware platforms in Duta; the Sutra app mirrors it in `protocol.rs` /
  `skrit.ts`.

## Transports

The CMD protocol is transport-independent. A **dual-CDC** device (e.g. CH552)
gives two USB pipes — a raw DATA console and a framed CMD port. A **single-channel**
device (one USB-CDC on ESP32 / Pico / nRF52840, or WebSocket) carries **both** over
one stream via **skrit-mux**: every frame is `COBS(channel, payload)`, channel 0 =
DATA, 1 = CMD. **BLE** is dual-channel by GATT design: a NUS service carries the raw
console and a sibling skrit CMD service carries the frames. A device sets the `muxed`
capability bit so the app wraps/unwraps instead of opening a second port. Network
transports are auth-gated (`AUTH`, factory default password). See
[PROTOCOL.md → Transports](PROTOCOL.md).

Implementing a new device = speak this protocol over whatever transport the
hardware has, answer `INFO` / `DEVICE_NAME` / `OUTPUT_DESC`, and you get the
whole Sutra app + its MCP server for free. (In Duta, the shared
`platforms/common` core already does all of this — you write only a thin HAL.)

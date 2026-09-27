# Pico 2 W Switch Controller Firmware

This standalone firmware makes a Raspberry Pi Pico 2 W enumerate as a wired
Nintendo Switch Pro Controller over USB. It runs a private Wi-Fi access point
and accepts short controller actions over UDP, so a host computer can drive the
controller without Bluetooth or a USB-UART adapter.

## Hardware path

```text
Host computer -- Wi-Fi --> Pico 2 W -- USB device --> Switch dock USB host
```

The UDP listener is `192.168.4.1:8765`. The action payload is JSON:

```json
{
  "action": {
    "buttons": ["A", "ZR"],
    "left_stick": {"x": 0.8, "y": 0.0},
    "right_stick": {"x": 0.0, "y": -0.25},
    "duration_ms": 120
  },
  "metadata": {"source": "host"}
}
```

Supported buttons are `A`, `B`, `X`, `Y`, `L`, `R`, `ZL`, `ZR`, `PLUS`,
`MINUS`, `HOME`, `CAPTURE`, `L3`, `R3`, `UP`, `DOWN`, `LEFT`, `RIGHT`, and
`WAIT`. Multiple entries in `buttons` are held at the same time. Stick axes
are normalized to `-1..1`, with positive `y` meaning up. The firmware resets
all buttons and sticks to neutral after `duration_ms` (maximum 10 seconds).

The Switch Pro HID report also has motion and rumble fields, but this Pico 2 W
build has no IMU and does not implement rumble feedback. Motion remains neutral
and rumble is ignored until additional hardware and protocol support are added.

For a direct smoke test after connecting the host to the Pico access point:

```sh
printf '%s' '{"action":{"buttons":["A","ZR"],"left_stick":{"x":0.5,"y":0},"duration_ms":120}}' \
  | nc -u -w1 192.168.4.1 8765
```

The binary is built for `PICO_BOARD=pico2_w` with the `WIFI` input backend.

## Upstream attribution

The USB Switch Pro controller emulation layer is based on the open-source
[`switch-pico`](https://github.com/jyapayne/switch-pico) project by Joey
Yakimowich-Payne. That upstream layer remains covered by its MIT license in
[`LICENSE`](LICENSE).

The Wi-Fi action backend, DHCP/access-point setup, UDP JSON protocol, and other
original additions in this repository are also released under the MIT license.

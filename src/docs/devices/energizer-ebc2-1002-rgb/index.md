---
title: "Energizer EBC2-1002-RGB"
date-published: 2026-08-13
type: light
standard: us
board: bk72xx
difficulty: 4
---

This is an Energizer BR30 RGBWW light bulb for the US market (E26 base). I found this as a 2-pack for $5 at BJ's.

## Initial Install

You must open the bulb to access the VDD, GND, RX, and TX pins. They can be accessed on the custom BK7231N daughterboard.

#### Initial Dissasembly
Use a chisel and hammer to separate the diffuser from the housing.

Use a thin, flathead screwdriver to pry the LED board up from the metal housing. This will disconnect the wires leading to the E26 socket. I reattached them by soldering new wires to the E26 socket and board. To avoid this, you can try pushing out the wires from these center two pins before prying up the board, using a needle or 22 gauge wire:

<img width="836" height="866" alt="image" src="https://github.com/user-attachments/assets/8ddae0d5-d47d-45bf-9f8b-2ea2d08cb9ca" />

(I soldered these pins for added rigidity, they are press-fit in the original assembly).

#### Flashing

Solder 3.3V, RX, TX, and GND to a USB to UART adapter from the pins on the bottom daughterboard. The pins are labelled in the silk screen.

<img width="1023" height="871" alt="image" src="https://github.com/user-attachments/assets/fe061c8e-bead-4532-abdb-8107f408f586" />

Solder a small jumper wire to the CEN pad, as you will need to temporarily jump this pin to GND when flashing.

Compile the ESPHome image and save it as a uf2 file.

Install ltchiptool on your computer, plug the UART adapter in, and run `ltchiptool flash write firmware.uf2`. Keep CEN shorted to GND for the first few seconds, as ltchiptool attempts to connect to the chip. Once you see a progress bar of flashing, disconnect the CEN pin.

Once your device fully flashes, reboots, and connects to the Wi-Fi, you can desolder the UART adapter, and re-assemble the device.

## Notes

Instead of using PWM, this device uses a bp5758 i2c 5-channel LED dimmer.

There are two on the main board, but only one is wired (5 channels - red, green, blue, warm white, cool white). The other one seems to be for a different SKU of product, as its traces lead nowhere on the board.

## Basic Configuration

```yaml
bk72xx:
  board: generic-bk7231n-qfn32-tuya

logger:

api:
  encryption:
    key: API_KEY

esphome:
  project:
    name: "Energizer.EBC2-1002 (RGBWW BR30)"
    version: "1.0.0"
  name: energizer-color-bulb-1
  friendly_name: ${display_name}

# Basic Config
substitutions:
  display_name: "Energizer Color Bulb 1"

ota:
  - platform: esphome
    password: !secret my_esp_ota_pwd

wifi:
  ssid: WIFI_USERNAME
  password: WIFI_PASSWORD
  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap: {} 

captive_portal:

web_server:
  port: 80
  local: True

sensor:
  - platform: wifi_signal
    name: "Wifi Signal Strength"
    update_interval: 15s
    disabled_by_default: true

time:
  - platform: homeassistant
    id: esptime

bp5758d:
  data_pin: 24
  clock_pin: 26

output:
  - platform: bp5758d
    id: output_red
    current: 15
    channel: 1
  - platform: bp5758d
    id: output_green
    current: 15
    channel: 2
  - platform: bp5758d
    id: output_blue
    current: 15
    channel: 3
  - platform: bp5758d
    id: output_cold_white
    current: 35
    channel: 4
  - platform: bp5758d
    id: output_warm_white
    current: 35
    channel: 5

# Tie the channels together into a single RGBW light entity
light:
  - platform: rgbww
    restore_mode: RESTORE_DEFAULT_OFF
    name: None
    red: output_red
    green: output_green
    blue: output_blue
    cold_white: output_cold_white
    warm_white: output_warm_white
    cold_white_color_temperature: 6500K
    warm_white_color_temperature: 2700K
```

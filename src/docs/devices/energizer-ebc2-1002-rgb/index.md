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
Use a chisel and hammer to separate the diffuser from the 

## GPIO Pinout

| Pin    | Function                           |
| ------ | ---------------------------------- |
| GPIO13 | Push Button (HIGH = off, LOW = on) |
| GPIO4  | Relay                              |
| GPIO15 | RED LED (HIGH = on, LOW = off)     |
| GPIO12 | BL0937 SEL                         |
| GPIO5  | BL0937 CF                          |
| GPIO14 | BL0937 CF1                         |
|        | Blue LED                           |

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

---
title: "Smartening up a dumb light switch to control smart bulbs"
date: 2019-06-01 12:00:00 -0300
last_modified_at: 2019-12-11 12:00:00 -0300
tags: [Home Assistant, ESPHome, LIFX, Shelly]
permalink: /smartening-up-a-dumb-light-switch-to-control-smart-bulbs/
---
<!-- markdownlint-disable html -->
I see a lot of people having issues and/or doubts about the light switches when installing smart bulbs in their homes. This is how I do it in my house.

As I mentioned in a previous post, I have several LIFX installed in the house. And, using only the original switches with them was out of the question. If a switch were turned off, the functionalities of the bulbs connected to that circuit would be lost.

On the other hand, I had to keep the switches for my older relatives (and some visitors), who always come here, and are used to using the "good old switches" or are not comfortable using voice assistants.

At first, I tried to use [Amazon Dash Buttons](/using-node-red-to-capture-dash-button-press/) while I kept the light button on. It worked very well, but the time between pushing the button and the light turn on was a bit annoying. Not to mention the complaints of those who used them: *"Why do your lights take so long to turn on?".* Sometimes it would take almost 3 seconds for a light to come on.

I also tried replacing the switches with TP-Link HS200 Smart Switches. They look good and fit very well on the wall, but they're just expensive relays and would not solve the issue. When switched off they also turned off the bulbs. It would need a custom firmware, but not all smart switches allow us to change their firmware.

Then I watched [this video](https://www.youtube.com/watch?v=J20hxfUTP9I) from *The Hook Up* talking about the [Shelly 1](https://shelly.cloud/shelly1-open-source/). That's what I needed.

I bought some units and flashed them with [ESPHome](https://esphome.io/).

<br />

## Sample config

ESPHome config for the living room switch:

```yaml
esphome:
  name: shelly01
  platform: ESP8266
  board: esp01_1m
  on_boot:
    priority: -10
    then:
      - switch.turn_on: relay

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

logger:

api:

ota:

binary_sensor:
  - platform: gpio
    pin: GPIO5
    name: "Living Room Light Switch"

switch:
  - platform: gpio
    name: "Living Room Light Relay"
    pin: GPIO4
    id: relay
```

<br />

The living room lights:

```yaml
light:
  - platform: group
    - name: "Living Room Ceiling Lights"
      entities:
        - light.living_room_ceiling1
        - light.living_room_ceiling2
```

<br />

I use Node-RED, so I'm a bit rusty in YAML. The automation would look something like this:

```yaml
automation:
- alias: 'Living Room Lights Turn ON'
  trigger:
    platform: state
    entity_id: binary_sensor.living_room_light_switch
  condition:
    condition: state
    entity_id: light.living_room_ceiling_lights
    state: 'off'
  action:
    service: light.turn_on
    data:
      entity_id: light.living_room_ceiling_lights
      kelvin: 7000
      brightness_pct: 100

- alias: 'Living Room Lights Turn OFF'
  trigger:
    platform: state
    entity_id: binary_sensor.living_room_light_switch
  condition:
    condition: state
    entity_id: light.living_room_ceiling_lights
    state: 'on'
  action:
    service: light.turn_off
    data:
      entity_id: light.living_room_ceiling_lights
```

<br />

## The pros

- **Really fast response** - It's like turning on and off an ordinary light bulb. Most people would not even notice that they are connected devices.
- **Smart bulbs always on** - You have the smart bulbs (and their features) available all the time.
- **Prevents smart bulbs resets** - Some smart bulbs are reset to factory defaults if turned on and off 5 or 6 times in a row. You can flick the light switch as much as you want and that won't happen, only the LEDs will be switched on and off. [Frenck would love this part!](https://youtu.be/orZ2xlH81KQ?t=3789) 😄.

<br />

## The cons

- **Additional cost** - Not really a con since they are not expensive. But needs to be taken into account.
- **Relay always on** - This worries me a bit because I think this can shorten its life span.
- **Connectivity** - Home Assistant needs to be accessible by the switches all the time.

<br />

I have been using this solution for a few months and am very happy with it, especially since I have had no issues so far.

<br />

## Improvements

[Brian Hanifin](https://brianhanifin.com/) has made amazing improvements to the setup above, including "*failover code which turns the smart switch back into a dumb switch should Home Assistant be unavailable to turn on the light when flipped*". All described in detail in his [blog post](https://brianhanifin.com/posts/esphome-shelly1-dumb-light-switch-smart/).

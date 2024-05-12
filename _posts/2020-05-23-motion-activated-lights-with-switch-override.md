---
title: "Motion-activated lights with switch override in just eight Node-RED nodes"
date: 2020-05-23 12:00:00 -0300
permalink: /motion-activated-lights-with-switch-override/
tags: [Home Assistant, Node-RED]
---
<!-- markdownlint-disable html -->
How to activate a light with a motion sensor and override it with a switch? This is a recurring subject on the [Forums](https://community.home-assistant.io), [Reddit](https://www.reddit.com/r/homeassistant/), and [Discord](https://discord.com/channels/330944238910963714/442034406073565204). Here's how I do it.

The light switch in my kitchen is connected to a [Shelly 1](https://shelly.cloud/shelly1-open-source/) flashed with [ESPHome](https://esphome.io/) with the configuration below.

```yaml
esphome:
  name: shelly1
  platform: ESP8266
  board: esp01_1m
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
ota:
  password: !secret esphome_ota_password
logger:
api:
  reboot_timeout:
    hours: 1
binary_sensor:
  - platform: gpio
    pin: GPIO5
    name: Kitchen Light Switch
    on_state:
      - switch.toggle: relay
switch:
  platform: gpio
  name: Kitchen Light Relay
  id: relay
  pin: GPIO4
```

<br />

When I flick the light switch, the relay in the Shelly 1 toggles, turning the light on or off.

Some time ago, I got a [Wyze Sense Starter Kit](https://wyze.com/wyze-sense.html). This kit comes with a small motion sensor that I installed in a strategic place in the kitchen, added it to Home Assistant using this [custom component](https://github.com/kevinvincent/ha-wyzesense) created by [Kevin Vincent](https://github.com/kevinvincent), and then I created this Node-RED flow.

![](/assets/img/Screen Shot 2020-05-21 at 11.08.09 AM.png)

The JSON code of this flow is available <a href="/files/motion-activated-lights-with-switch-override.json" download target="_blank">here</a>.

<br />

By default, when motion is detected, the light turns on automatically. The light goes off after 10 minutes if no movement is detected during that period. If a movement is detected while the light is still on, the 10-minute countdown is restarted.

If the light is turned on using the wall switch, the node that monitors the motion sensor is deactivated, and the light stays on until it is turned off using the wall switch. After it's turned off, the node that monitors the motion sensor is re-activated, and the rule described above works again.

<br />

## Bonus

Adding one more node to the flow you can turn off the light from the frontend and it will re-enable the trigger node (if needed) and reset the current countdown.

![](/assets/img/Screen Shot 2020-05-21 at 11.08.21 AM.png)

The JSON code of this flow is available <a href="/files/motion-activated-lights-with-switch-override-bonus.json" download target="_blank">here</a>.

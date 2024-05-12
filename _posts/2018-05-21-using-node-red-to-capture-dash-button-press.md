---
title: "Using Node-RED to capture Dash Button press"
date: 2018-05-21 12:00:00 -0300
last_modified_at: 2018-05-28 12:00:00 -0300
tags: [Home Assistant, Node-RED]
permalink: /using-node-red-to-capture-dash-button-press/
---
<!-- markdownlint-disable html -->
> With the release of Home Assistant version 0.70 it is now possible to install the Dasher add-on again when running Hass.io on Ubuntu Server.
{: .prompt-info }

If, like me, you are frustrated by the inability to install certain add-ons in the latest versions of [Hass.io running on a generic Linux Server](https://www.home-assistant.io/hassio/installation/#alternative-install-on-generic-linux-server), you must be looking for alternatives.

One of the add-ons I was missing the most was [Dasher](https://github.com/james-fry/hassio-addons/tree/master/dasher) because I use [Dash Buttons](https://www.amazon.com/ddb/learn-more) to control some scenes in my house.

As I do not have the patience to wait for the next versions of <s>Hass.io</s> the add-on \(which may or may not solve the problem\) I started searching and found [Amazon Dash Button with Node-RED built-in nodes](https://solarhinted.blogspot.com/2017/07/amazon-dash-button-with-node-red-built.html).

This article explains everything textually so I decided to prepare something more visual to make it easier to understand it.

Based on what's written there I created the following subflow.

<br />

![Subflow](/assets/img/Screenshot 2018-05-21 18.28.57.png)

<br />

```json
[
    {
        "id": "27274ec0.477d32",
        "type": "subflow",
        "name": "Dash Button Press",
        "info": "",
        "in": [],
        "out": [
            {
                "x": 460,
                "y": 40,
                "wires": [
                    {
                        "id": "4d2c0a32.de2794",
                        "port": 0
                    }
                ]
            },
            {
                "x": 460,
                "y": 120,
                "wires": [
                    {
                        "id": "4d2c0a32.de2794",
                        "port": 1
                    }
                ]
            }
        ]
    },
    {
        "id": "10f156e0.c68be9",
        "type": "udp in",
        "z": "27274ec0.477d32",
        "name": "",
        "iface": "",
        "port": "67",
        "ipv": "udp4",
        "multicast": "false",
        "group": "",
        "datatype": "buffer",
        "x": 70,
        "y": 80,
        "wires": [
            [
                "94bc49a4.4eb12"
            ]
        ]
    },
    {
        "id": "94bc49a4.4eb12",
        "type": "function",
        "z": "27274ec0.477d32",
        "name": "",
        "func": "var mac = Buffer.alloc(6);\nmsg.payload.copy(mac, targetStart=0, sourceStart=28, sourceEnd=34);\nmsg.mac = mac.toString('hex');\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "x": 210,
        "y": 80,
        "wires": [
            [
                "4d2c0a32.de2794"
            ]
        ]
    },
    {
        "id": "4d2c0a32.de2794",
        "type": "switch",
        "z": "27274ec0.477d32",
        "name": "",
        "property": "mac",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "78e103b8b5c8",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "78e1031a7c50",
                "vt": "str"
            }
        ],
        "checkall": "false",
        "repair": false,
        "outputs": 2,
        "x": 350,
        "y": 80,
        "wires": [
            [],
            []
        ]
    }
]
```

<br />

The switch node has two outputs, one for each one of the buttons I use. You'll need to have as many outputs as the quantity of buttons you want to use.

<br />

![Switch Node](/assets/img/Screenshot 2018-05-21 18.29.18.png)

<br />

After that I just created the following flow connecting each output to its respective node and deploy the flows.

<br />

![Flow](/assets/img/Screenshot 2018-05-21 18.29.45.png)

<br />

If you're curious, these are the switches I currently use.

<br />

![Dash - Bedroom Lights](/assets/img/Screenshot 2018-05-21 18.29.48.png)

<br />

{% raw %}

```yaml
switch:
  - platform: template
    switches:
      dash_bedroom:
        value_template: "{{ is_state('light.bedroom_lights', 'on') }}"
        turn_on:
          service: script.turn_on
          entity_id: script.good_morning
        turn_off:
          service: script.turn_on
          entity_id: script.good_night
```

{% endraw %}

<br />

![Dash - Livingroom Lights](/assets/img/Screenshot 2018-05-21 18.29.53.png)

<br />

{% raw %}

```yaml
switch:
  - platform: template
    switches:
      living_room_lights:
        value_template: "{{ is_state('light.living_room_lights', 'on') }}"
        turn_on:
          service: script.turn_on
          entity_id: script.turn_on_1s
        turn_off:
          service: script.turn_on
          entity_id: script.turn_off_1s
```

{% endraw %}

<br />

The best thing about this is that it seems to work faster than with the add-ons in Home Assistant.

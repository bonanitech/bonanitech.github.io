---
title: "Blink lights when your IP phone rings"
date: 2018-03-21 12:00:00 -0300
tags: [Home Assistant, FreePBX, Asterisk]
permalink: /blink-lights-when-your-ip-phone-rings/
---
<!-- markdownlint-disable html -->
I have an IP phone on my desk connected to a [FreePBX](https://www.freepbx.org) server. It is permanently set to silent mode. I keep it that way because 90% of the time I work with headphones. It's also in a place where I can not easily see it.

So how do I know when the phone is ringing? Easy. The lights flash in a different color from the ambient light, in this case Navy Blue.

And how did I set this up? Also easy.

On the FreePBX server, all incoming calls are redirected to a Ring Group containing two extensions:

* My desk phone - extension 200
* "Fake" extension - extension \*\*\*\*5678

<br />

![FreePBX Ring Group]({{ "/assets/img/Screenshot-2018-03-21-13.47.34.png" | absolute_url }})

<br />

The \*\*\*\*5678 extension is created in the `extensions_custom.conf` file.

```asterisk
[from-internal-custom]
exten => ****5678,1,TrySystem(/home/asterisk/hass.sh > /dev/null 2>&1 &)
same => n,Congestion
```

<br />

It calls the `/home/asterisk/hass.sh`{: .filepath} shell script.

```bash
#!/bin/bash

curl -X POST -H "x-ha-access: HASS_PASSWORD" -H "Content-Type: application/json" http://HASS_SERVER_ADDRESS:8123/api/services/script/incoming_call
```
{: file="/home/asterisk/hass.sh" }

<br />

Which triggers the `incoming_call` script in Home Assistant.

```yaml
# FreePBX Incoming Call
incoming_call:
  sequence:
    - service: light.lifx_effect_pulse
      data:
        entity_id: light.lamp_lights
        color_name: navy
        brightness: 191
        period: 1
        cycles: 2
    - delay:
        seconds: 4
    - service: light.lifx_effect_pulse
      data:
        entity_id: light.lamp_lights
        color_name: navy
        brightness: 191
        period: 1
        cycles: 2
    - delay:
        seconds: 4
    - service: light.lifx_effect_pulse
      data:
        entity_id: light.lamp_lights
        color_name: navy
        brightness: 191
        period: 1
        cycles: 2
```

<br />

When a call is received, the phone "rings" silently and the lights blink in navy blue three times separated by delays of 4 seconds.

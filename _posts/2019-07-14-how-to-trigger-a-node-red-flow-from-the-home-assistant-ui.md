---
title: "How to trigger a Node-RED flow from the Home Assistant UI"
date: 2019-07-14 12:00:00 -0300
tags: [Home Assistant, Node-RED]
permalink: /how-to-trigger-a-node-red-flow-from-the-home-assistant-ui/
---
<!-- markdownlint-disable html -->
I see a lot of people who use Node-RED as their automation engine looking for a way to trigger a flow from the Home Assistant UI. Here's how I do it.

First I created an empty `script`.

```yaml
script:
  good_morning:
    sequence:
```

<br />

And a button in Lovelace.

```yaml
- type: entity-button
  entity: script.good_morning
  name: "Good Morning"
  icon: mdi:coffee
  tap_action:
    action: call-service
    service: script.turn_on
    service_data:
      entity_id: script.good_morning
```

<br />

Then in Node-RED.

<br />

![flow](/assets/img/2019-07-14-flow.png)

<br />

The `events: all` node checks only events of type `call_service`.

<br />

![events](/assets/img/2019-07-14-events.png)

<br />

The `switch` node checks which `script` was called.

<br />

![switch](/assets/img/2019-07-14-switch.png)

<br />

The `call-service` node turns on my bedroom ceiling lights in a 30 seconds transition.

<br />

![call-service](/assets/img/2019-07-14-call-service.png)

<br />

When I press the button (below) in the UI or say "Hey Google, good morning" [^1] the flow is triggered and the lights turn on.

<br />

![button](/assets/img/2019-07-14-button.png)

<br />

I prefer this approach because I do not like the idea of using `input_boolean` for this. It would be like using a light switch to activate a doorbell.

[^1]: The script was exposed to Google Assistant using [Home Assistant Cloud](https://www.home-assistant.io/cloud/).

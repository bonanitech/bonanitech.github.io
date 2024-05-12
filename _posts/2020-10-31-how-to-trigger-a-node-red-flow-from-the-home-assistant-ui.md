---
title: "How to trigger a Node-RED flow from the Home Assistant UI (new version)"
date: 2020-10-31 12:00:00 -0300
last_modified_at: 2021-07-14 12:00:00 -0300
tags: [Home Assistant, Node-RED]
permalink: /how-to-trigger-a-node-red-flow-from-the-home-assistant-ui-new-version/
---
<!-- markdownlint-disable html -->
Until a few months ago, I was using [this method](https://bonani.tech/how-to-trigger-a-node-red-flow-from-the-home-assistant-ui/) to trigger a flow from the Home Assistant UI. Although it works fine, it’s not very elegant. Empty scripts cause linting warnings, and we don’t know if that will be forever supported by Home Assistant.

As [pointed out](https://bonani.tech/how-to-trigger-a-node-red-flow-from-the-home-assistant-ui/#comment-4794636679) by iantrich (a long time ago! 😳) in the comments section of my old post, since version [0.20.0](https://github.com/zachowj/node-red-contrib-home-assistant-websocket/releases/tag/v0.20.0) of `node-red-contrib-home-assistant-websocket`, we can "Trigger an exposed event node from a service call".

> With the release of the Node-RED [custom component](https://github.com/zachowj/hass-node-red) version 0.3.0, it adds the ability to trigger an event node from a service call.

That makes triggering a Node-RED flow from the Home Assistant UI even easier.

First, in Node-RED, we need to add an `entity` node to the flow.

<br />

![entity-node](/assets/img/2020-10-31-entity-node.png)

<br />

Change its type to switch, and add a name in the Home Assistant Config section.

<br />

![edit-entity-node](/assets/img/2020-10-31-edit-entity-node.png)

<br />

Then we can connect our flow to the first output of the node.

<br />

![flow](/assets/img/2020-10-31-flow.png)

<br />

After that, in Home Assistant, we add the following code to a Lovelace view.

```yaml
- type: button
  entity: script.good_morning
  name: "Good Morning"
  icon: mdi:coffee
  tap_action:
    action: call-service
    service: nodered.trigger
    service_data:
      entity_id: switch.scene_good_morning
```

> `script.good_morning` is used here as the entity because I didn’t want the button to change its color. That script was created using the code [below](#exposing-to-voice-assistants). If you don't want or don't need to use that script, you can use `switch.scene_good_morning` as the entity here.
{: .prompt-info }

<br />

This will create the button below.

<br />

![button](/assets/img/2019-07-14-button.png)

<br />

And when we press the button, it will trigger the flow we created above.

<br />

## Exposing to voice assistants

If we want to use our favorite voice assistant to trigger that flow, we can create a script.

```yaml
good_morning:
  alias: Good Morning
  sequence:
  - service: nodered.trigger
    data: {}
    entity_id: switch.scene_good_morning
  mode: single
```

And then we expose it using [Home Assistant Cloud](https://www.home-assistant.io/cloud/) or other method.

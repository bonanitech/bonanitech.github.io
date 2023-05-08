---
title: "Making Home Assistant's Presence Detection not so Binary (Node-RED version)"
date: 2018-11-14 12:00:00 -0300
last_modified_at: 2022-04-05 12:00:00 -0300
tags: [Home Assistant, Node-RED]
permalink: /making-home-assistants-presence-detection-not-so-binary-node-red-version/
---
<!-- markdownlint-disable html -->
My special thanks to Phil Hawthorne for letting me use the title of his post here.

In early 2018 I found Phil's excellent blog post "[Making Home Assistant’s Presence Detection not so Binary](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/)" and began using the setup suggested by him almost immediately.

A few months later I started migrating my automations to Node-RED and posted the initial results in the [comments section](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/#comment-3945751887) of Phil's post.

This works great, but the caveat is that you have to have one sequence for each device tracked.

I started looking for a way to have only one sequence for all the tracked devices, after some time (with some hits and many, many misses) this is what I came out with.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v1.json" download target="_blank">here</a>.

<br />

Now all I needed was to set up an `input_select` with the same name of a `device_tracker` in Home Assistant.

For reference, in `known_devices.yaml`.

```yaml
homer:
  hide_if_away: false
  icon:
  mac: 00:00:00:00:00:00
  name: Homer
  picture: /local/homer.png
  track: true
```

> The `known_devices.yaml` file was [removed](https://www.home-assistant.io/blog/2019/06/05/release-94/#modernizing-the-device-tracker) from Home Assistant in version 0.94. It's suggested to use the person integration instead.
{: .prompt-danger }

<br />

Then I added this to `configuration.yaml`.

```yaml
input_select:
  homer:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
```

<br />

And in Node-RED an `events: state` node that should be connected to the change node in the beginning of the sequence.

<br />

![Homer]({{ "/assets/img/2018-11-14-homer_node.png" | absolute_url }})

<br />

This must be repeated for each person being tracked, just replacing the names where needed.

For example:

`known_devices.yaml`

```yaml
marge:
  hide_if_away: false
  icon:
  mac: 11:11:11:11:11:11
  name: Marge
  picture: /local/marge.png
  track: true
```

<br />

`configuration.yaml`

```yaml
input_select:
  marge:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
```

<br />

![Marge]({{ "/assets/img/2018-11-14-marge_node.png" | absolute_url }})

<br />

The final result should be something like this:

<br />

![Node-RED]({{ "/assets/img/2018-11-14-node-red.png" | absolute_url }})

<br />

![Home Assistant]({{ "/assets/img/2018-11-14-home-assistant.png" | absolute_url }})

<br />

## Update - Dec 25, 2018

If you use `node-red-contrib-home-assistant-websocket` (default in Node-RED Community Hass.io Add-on), the sequence can be even smaller.

Since its [version 0.3.0](https://github.com/zachowj/node-red-contrib-home-assistant-websocket/releases/tag/v0.3.0):

> Call Service and Fire Event nodes are now able to render mustache templates in the data property.

This means that we can remove almost all of the template nodes used in the original sequence and move their content to the call service nodes right after them.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v2.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v2.json" download target="_blank">here</a>.

<br />

## Update - Nov 04, 2019

Here's the most recent version of the sequence. Now I'm using the `person` integration to track people. If you're using `device_tracker`, remember to change the first and third nodes according to your needs.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v3.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v3.json" download target="_blank">here</a>.

<br />

## Update - May 01, 2020

A reader made me aware of an issue in the sequence when using the `zone` integration in Home Assistant. The tracked person's state would change to "Just Left" again when entering or exiting a zone.

Here's the sequence with a fix for this issue.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v4.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v4.json" download target="_blank">here</a>.

Thank you, [CamelY0](https://disqus.com/by/camely0/), for finding, reporting, and helping to fix the issue.

<br />

## Update - Jun 17, 2020

This sequence went through a diet. Now we no longer need the "rbe" and "template" nodes in it, nor do we need to use the "change" nodes to reset the timers.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v5.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v5.json" download target="_blank">here</a>.

<br />

## Update - Jun 17, 2020 (2)

Always the `zones`! 🤦🏼‍♂️

The issue mentioned on May 01, 2020 (see [above](#update---may-01-2020)) found its way in again. Here's the sequence with the fix.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v6.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v6.json" download target="_blank">here</a>.

My thanks to Kermit [Jason Zachow](https://github.com/zachowj) for the JSONata tips.

<br />

## Update - Apr 05, 2022

I was thinking the other day, "_What if I go on vacation and set my status to Extended Away right after I leave home? This way I would force the house into Vacation Mode_".

When I tried doing this, I soon noticed a problem. If I change my status to _Extended Away_ less than 10 minutes after leaving home, my status will change to _Away_ because the current flow does not account for manual status changes.

To fix that, I created this new flow (or set of flows).

![Template Sequence]({{ "/assets/img/2022-05-05-template_sequence_v7.png" | absolute_url }})

The JSON code of this sequence is available <a href="/files/making-home-assistants-presence-detection-not-so-binary-node-red-version-v7.json" download target="_blank">here</a>.

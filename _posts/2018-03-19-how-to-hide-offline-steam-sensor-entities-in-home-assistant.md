---
title: "How to hide offline Steam sensor entities in Home Assistant"
date: 2018-03-19 12:00:00 -0300
last_modified_at: 2020-02-05 12:00:00 -0300
tags: [Home Assistant]
permalink: /how-to-hide-offline-steam-sensor-entities-in-home-assistant/
---
<!-- markdownlint-disable html -->
> The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, the code below won't work anymore after that. Follow [this link](#lovelace-version) for the Lovelave version.
{: .prompt-danger }

I received the message below on [Reddit](https://www.reddit.com/r/homeassistant/comments/85fbob/managing_groups_visibility_in_home_assistant/dvx0473/) and decided to find a way to do what was proposed.

>Hi, this is great thanks so much for your contributions to the community. Blogs like yours are the reason I got in to HA in the first place.  
>I’d be interested to see if you find a way to hide individual sensors from appearing groups. For example I have a group of steam sensors in a group called steam and would love to hide them individually when someone is not online without having to hide the whole group.

A few hours later...

```yaml
sensor:
  - platform: steam_online
    api_key: !secret steam_api_key
    accounts:
      - 12345678901234567
      - 98765432109847553
      - 98409840789049048
      - 90848949084989804
```

<br />

## Update - Jun 18, 2018

After a request here on the comments section and on [Reddit](https://www.reddit.com/r/homeassistant/comments/85fbob/managing_groups_visibility_in_home_assistant/e0t40up/) I found out that, if the last or all entities of the sensor list in the automation template are offline, it would cause an error when the automation is run and therefore the frontend would not display the correct information in the group card.

At first I did not know how to solve this and suggested to the requester that he tried to use [CustomUI](https://github.com/andrey-git/home-assistant-custom-ui/blob/master/docs/templates.md#make-a-group-that-contains-all-on-entities) as an alternative.

As the problem did not leave my head, I challenged myself to solve it. Then, after much trial and error, I came out with the following automation. It solves the problem of when the last entity is offline, and the group (`group.steam`) is not automatically created when all entities are offline.

It also solves another problem I encountered during the tests. If only one entity is online (not the last), the group is now created correctly.

I really did not like the `if is_state_attr(steam.entity_id, 'icon', 'mdi:steam')` part, but I did not find another way to select only the sensor.steam_* entities. If anyone knows a better way to do it please tell us how on the comments section below.

> I'm keeping the old automation (commented out) below the current one for readers to understand the context.
{: .prompt-info }

{% raw %}

```yaml
automation:
- alias: 'Group Entities Online/Offline'
  trigger:
    - platform: homeassistant
      event: start
    - platform: time
      minutes: '/1'
      seconds: 0
  action:
    - service: group.set
      data_template:
        object_id: steam
        entities: "{% set counter = 0 %}
                {%- for steam in states.sensor if is_state_attr(steam.entity_id, 'icon', 'mdi:steam') and not(is_state(steam.entity_id, 'offline')) -%}
                {% set counter = counter + 1 %}
                {%- endfor -%}
                {%- for steam in states.sensor if is_state_attr(steam.entity_id, 'icon', 'mdi:steam') and not(is_state(steam.entity_id, 'offline')) -%}
                {%- if loop.first -%}{%- if counter == 1 -%}{{ steam.entity_id | lower }}{%- else -%}{{ steam.entity_id | lower }},{%- endif -%}{%- elif loop.last -%}{{ steam.entity_id | lower }}{%- else -%}{{ steam.entity_id | lower }},{%- endif -%}
                {%- endfor -%}"

#   - alias: 'Steam Group Entities 1'
#     trigger:
#       - platform: homeassistant
#         event: start
#     action:
#       - service: group.set
#         data_template:
#           object_id: steam
#           entities: "{% if not(is_state('sensor.steam_12345678901234567', 'offline')) %}sensor.steam_12345678901234567,{% endif %}
#           {% if not(is_state('sensor.steam_98765432109847553', 'offline')) %}sensor.steam_98765432109847553,{% endif %}
#           {% if not(is_state('sensor.steam_98409840789049048', 'offline')) %}sensor.steam_98409840789049048,{% endif %}
#           {% if not(is_state('sensor.steam_90848949084989804', 'offline')) %}sensor.steam_90848949084989804{% endif %}"
#
#   - alias: 'Steam Group Entities 2'
#     trigger:
#       - platform: time
#         minutes: '/1'
#         seconds: 0
#     action:
#       - service: group.set
#         data_template:
#           object_id: steam
#           entities: "{% if not(is_state('sensor.steam_12345678901234567', 'offline')) %}sensor.steam_12345678901234567,{% endif %}
#           {% if not(is_state('sensor.steam_98765432109847553', 'offline')) %}sensor.steam_98765432109847553,{% endif %}
#           {% if not(is_state('sensor.steam_98409840789049048', 'offline')) %}sensor.steam_98409840789049048,{% endif %}
#           {% if not(is_state('sensor.steam_90848949084989804', 'offline')) %}sensor.steam_90848949084989804{% endif %}"
```

{% endraw %}

<br />

**It works!**

Some important points:

- Pay attention to the commas after each sensor in the templates, the last doesn't need one.
- If you use views, don't forget to add the group (in this case `group.steam`) to the desired view.
- You don't need to create the group, it will be created by the automations.

<br />

## Lovelace version

Now with the new Lovelace UI it's even easier to do it. We just need to use this `entity-filter` in `ui-lovelace.yaml`.

```yaml
- type: entity-filter
  entities:
    - sensor.steam_12345678901234567
    - sensor.steam_98765432109847553
    - sensor.steam_98409840789049048
    - sensor.steam_90848949084989804
  state_filter:
    - 'online'
    - 'busy'
    - 'away'
    - 'snooze'
    - 'looking_to_trade'
    - 'looking_to_play'
  card:
    type: glance
    title: Steam
  show_empty: false
```

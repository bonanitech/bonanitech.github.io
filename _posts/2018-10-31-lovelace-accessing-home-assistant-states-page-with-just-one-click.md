---
title: "[Lovelace] Accessing Home Assistant states page with just one click"
date: 2018-10-31 12:00:00 -0300
last_modified_at: 2020-02-05 12:00:00 -0300
tags: [Home Assistant, Lovelace]
permalink: /lovelace-accessing-home-assistant-states-page-with-just-one-click/
---
<!-- markdownlint-disable html -->
> The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that.
{: .prompt-danger }

Sometimes you need to access the Home Assistant `states` page \(old UI\) and when you have Lovelace as the default UI you have to type it in the address bar. Here is an easier way to access it.

Add the following code to your `configuration.yaml` file and restart Home Assistant.

```yaml
panel_iframe:
  states:
    title: "States"
    icon: mdi:home-assistant
    url: http://<your_home_assistant_url>/states
```

> Make sure you have the correct value in `url:`
{: .prompt-warning }

<br />

Now you can access the `states` page with just one click.

<br />

## Update - Dec 24, 2018

An alternative way to achieve this is to place a `weblink` in a Lovelace entities card.

```yaml
- type: entities
  show_header_toggle: false
  entities:
    - type: weblink
      url: /states
      name: "States"
      icon: mdi:home-assistant
```

<br />

![States](/assets/img/2018-10-31-states.png)

---
title: "Dynamically hide/unhide an entire view in Home Assistant"
date: 2018-03-15 12:00:00 -0300
last_modified_at: 2020-02-05 12:00:00 -0300
tags: [Home Assistant]
permalink: /dynamically-hide-unhide-an-entire-view/
---
<!-- markdownlint-disable html -->
> The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that.
{: .prompt-danger }

Today I learned a way to hide and unhide a view in Home Assistant. It turns out it’s easier than I expected. All I had to do was use the `group.set`  service to change the attribute `view`.

Here is an example code:

```yaml
group:
  default_view:
    name: "First"
    view: yes
    entities:
      - script.show_view
      - script.hide_view
  second_view:
    name: "Second"
    view: yes
    entities:
      - script.show_view
      - script.hide_view

script:
  show_view:
    alias: "Show Second View"
    sequence:
      - service: group.set
        data:
          object_id: second_view
          view: true
  hide_view:
    alias: "Hide Second View"
    sequence:
      - service: group.set
        data:
          object_id: second_view
          view: false
```

<br />

This generates two views \(both visible\). When I call the \"Hide Second View\" script, the \"Second\" view become hidden. And when I call the \"Show Second View\" script, the \"Second\" view become visible again.

<br />

![]({{ "/assets/img/Screenshot-2018-03-15-20.25.38-e1521161151544.png" | absolute_url }})

<br />

![]({{ "/assets/img/Screenshot-2018-03-15-20.25.36-e1521161174731.png" | absolute_url }})

<br />

The only drawback is that if you only have two views and hide one of them, the blue header bar will be resized, but the placement of the content below it will not be updated automatically.

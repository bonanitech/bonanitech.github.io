---
title: "Track battery levels with Home Assistant and Custom UI"
date: 2018-04-25 12:00:00 -0300
last_modified_at: 2020-02-05 12:00:00 -0300
tags: [Home Assistant, CustomUI]
permalink: /track-battery-levels-with-home-assistant-and-custom-ui/
---
<!-- markdownlint-disable html -->
> The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that.
{: .prompt-danger }

If you have sensors around the house, you should be concerned about the battery life of these sensors. Unless they are hard wired, of course. 🙂

I was reading [this example](https://www.home-assistant.io/cookbook/track_battery_level/) from the [Home Assistant Cookbook](https://www.home-assistant.io/cookbook) and started to think *"why not change the colors of the entities according to their battery level?"*.

It turns out that there is no way to color entities only with Home Assistant, for that we will have to use [Custom UI](https://github.com/andrey-git/home-assistant-custom-ui). Its installation is quite simple, just follow one of the procedures described in [documentation](https://github.com/andrey-git/home-assistant-custom-ui/blob/master/docs/installing.md).

I chose to install it [manually](https://github.com/andrey-git/home-assistant-custom-ui/blob/master/docs/installing.md#manual-install) and activated using the code below like described in the [Activating](https://github.com/andrey-git/home-assistant-custom-ui/blob/master/docs/activating.md) section of the documentation.

```yaml
customizer:
  custom_ui: local
```

<br />

Then it was time to have fun with the YAML files.

```yaml
frontend:
  themes:
    alert_yellow:
      primary-text-color: '#FFC000'
      paper-item-icon-color: '#FFC000'
    alert_red:
      primary-text-color: '#FF0000'
      paper-item-icon-color: '#FF0000'
```

<br />

First I created two themes, `alert_yellow` and `alert_red`. Each of them define the colors for the text \(name and state\) and for the icon of a given entity.

{% raw %}

```yaml
sensor:
  - platform: template
    sensors:
      window_sensor_battery_level:
        unit_of_measurement: '%'
        value_template: '{{ states.input_number.window_sensor_battery.state|int }}'
        icon_template: >
          {% set battery_level = states.sensor.window_sensor_battery_level.state|default(0)|int %}
          {% set battery_round = (battery_level / 10) |int * 10 %}
          {% if battery_round >= 100 %}
            mdi:battery
          {% elif battery_round > 0 %}
            mdi:battery-{{ battery_round }}
          {% else %}
            mdi:battery-alert
          {% endif %}
      door_sensor_battery_level:
        unit_of_measurement: '%'
        value_template: '{{ states.input_number.door_sensor_battery.state|int }}'
        icon_template: >
          {% set battery_level = states.sensor.door_sensor_battery_level.state|default(0)|int %}
          {% set battery_round = (battery_level / 10) |int * 10 %}
          {% if battery_round >= 100 %}
            mdi:battery
          {% elif battery_round > 0 %}
            mdi:battery-{{ battery_round }}
          {% else %}
            mdi:battery-alert
          {% endif %}
```

{% endraw %}

<br />

Then created the sensors using the `icon_template` code found in the Cookbook to show the icon corresponding to each battery level.

I used two `input_number` to simulate the battery charge values of each `sensor`.

```yaml
customize:
  sensor.window_sensor_battery_level:
    custom_ui_state_card: state-card-custom-ui
    friendly_name: "Window Sensor Battery Level"
    templates:
      theme: >
        if (state < 20) {
          return 'alert_red';
        } else if (state < 40) {
          return 'alert_yellow';
        }
  sensor.door_sensor_battery_level:
    custom_ui_state_card: state-card-custom-ui
    friendly_name: "Door Sensor Battery Level"
    templates:
      theme: >
        if (state < 20) {
          return 'alert_red';
        } else if (state < 40) {
          return 'alert_yellow';
        }
```

<br />

And finally I customized each sensor. If the battery charge is below 40% the sensor will be displayed in yellow and if it’s below 20% it will be displayed in red. If the values are 40% and above the sensor will be displayed using the colors defined in the theme used at that moment.

Here are some samples of the output.

<br />

![Battery Levels](/assets/img/Screenshot 2018-04-25 11.26.52.png)

![Battery Levels](/assets/img/Screenshot 2018-04-25 11.26.58.png)

![Battery Levels](/assets/img/Screenshot 2018-04-25 11.27.09.png)

![Battery Levels](/assets/img/Screenshot 2018-04-25 11.27.16.png)

<br />

Using this with a good notification system will help prevent a sensor from stop responding due to a discharged battery. Additionally, this will give you the possibility to choose the best time to change each battery and minimize the risk of discarding batteries that can still be used for some time.

Unfortunately this is not working with Safari 11.1 on MacOS High Sierra, I had to use Chrome.

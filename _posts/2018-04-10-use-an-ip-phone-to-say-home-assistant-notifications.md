---
title: "Use an IP phone to 'say' Home Assistant notifications"
date: 2018-04-10 12:00:00 -0300
tags: [Home Assistant, FreePBX, Asterisk]
permalink: /use-an-ip-phone-to-say-home-assistant-notifications/
---
<!-- markdownlint-disable html -->
After I posted [Blink lights when your IP phone rings]({{ site.baseurl }}{% post_url 2018-03-21-blink-lights-when-your-ip-phone-rings %}), I kept thinking "Why not use my IP phone to 'say' notifications?"

The way I did it was creating a Paging Group on my FreePBX server.

<br />

![Paging and Intercom]({{ "/assets/img/Screenshot 2018-04-08 19.51.40.png" | absolute_url }})

<br />

![Page Group]({{ "/assets/img/Screenshot 2018-04-08 19.51.49.png" | absolute_url }})

<br />

Then in Home Assistant I have the following.

```yaml
shell_command:
  mauricio_arrived: printf 'Channel: Local/724464@from-internal\nApplication: Flite\nData: "*. Welcome home, Mauricio."' > `date +"%Y%m%d%H%M%S"`.call && scp *.call root@192.168.10.10:/var/spool/asterisk/outgoing && rm *.call

automation:
  - alias: Mark Mauricio as just arrived
    trigger:
      - platform: state
        entity_id: input_boolean.mauricio_home
        from: "off"
        to: "on"
    action:
      - service: shell_command.mauricio_arrived
```

> You must set SSH to use keys for authentication, this will prevent the system from prompting for passwords every time shell command is executed.
{: .prompt-tip }

<br />

It will generate a file with the content below and upload it via SCP to the directory `/var/spool/asterisk/outgoing`{: .filepath} on the FreePBX server. Asterisk uses files in this directory to initiate calls automatically, see [this](https://wiki.asterisk.org/wiki/display/AST/Asterisk+Call+Files) and [this](https://www.voip-info.org/wiki/view/Asterisk+auto-dial+out).

```yaml
Channel: Local/724464@from-internal
Application: Flite
Data: "*. Welcome home, Mauricio."
```

> I inserted `*.` in the text because my phone \(Sangoma S500\) has a two second beep that is played 'before' the audio, but it overlaps the beggining of the text being 'said'. I tried removing the beep in the phone configuration, but then I had a two second silence with the same problem.
{: .prompt-info }

<br />

When I get home, the automation is triggered and I'm greeted with a robotic voice. 🤖

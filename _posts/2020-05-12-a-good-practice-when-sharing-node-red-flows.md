---
title: "A good practice when sharing Node-RED flows with Home Assistant nodes"
date: 2020-05-12 12:00:00 -0300
tags: [Home Assistant, Node-RED]
permalink: /a-good-practice-when-sharing-node-red-flows-with-home-assistant-nodes/
---
<!-- markdownlint-disable html -->
So you've created an amazing flow, and now you want to share it with the world. How to minimize the chances of issues for those who are going to import this flow?

It's important to keep that in mind, especially when you are sharing flows with Home Assistant nodes. Nobody wants to end up with multiple server nodes by mistake.

![servers](/assets/img/2020-05-12-config-nodes.png)

<br />

## Before sharing

If you want to share a flow, it’s a good practice to remove the part that defines the Home Assistant configuration node and the lines that reference it.

[Jason Zachow](https://github.com/zachowj) (a.k.a. Kermit on Discord), the developer of [node-red-contrib-home-assistant-websocket](https://github.com/zachowj/node-red-contrib-home-assistant-websocket), created a tool that does that automatically for you. It also removes other info like latitude and longitude. It's available [here](https://zachowj.github.io/node-red-contrib-home-assistant-websocket/scrubber/). Thank you, Jason!

It’s very easy to use Jason’s tool. Just paste the JSON code you exported in the upper text box, click on the ‘Scrub’ button, and in the lower text box, it will return the shareable code.

<br />

## Importing flows

If you are importing flows from other people and you're not sure if the code is clean, you can also use the tool mentioned above.

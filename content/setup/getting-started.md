---
title: Getting started
weight: 1
---

1. Identify your prerequisites:
    - the datasource(s) to use and their type, and the data you want to capture on each. Once done, identify if the corresponding [datasource](../configuration/plugins/datasources/) plugin(s) exists
    - choose (and maybe install) the message bus you'll use
    - identify which [hermes-client](../configuration/plugins/hermes-client/) plugin(s) you'll need

2. Install Hermes by following the [Installation](../installation/) section

3. Configure hermes-server by following the following sections:
    - [hermes](../hermes/) for global settings
    - configure your [messagebus](../configuration/plugins/messagebus_producers/) plugin
    - configure your [datasource](../configuration/plugins/datasources/) plugin(s)
    - configure your [attribute](../configuration/plugins/attributes/) plugins, if any
    - [hermes-server](../configuration/hermes-server/) for hermes-server settings, and set up your server datamodel

4. Run hermes-server by following the [Run](../run/) section, and once it has successfully done its first data polling, generate an initsync sequence using the hermes-server CLI, as explained in the [Run](../run/) section

5. Configure a first hermes-client by following the following sections:
    - [hermes](../hermes/) for global settings
    - configure your [messagebus](../configuration/plugins/messagebus_consumers/) plugin
    - configure your [attribute](../configuration/plugins/attributes/) plugins, if any
    - configure your [hermes-client](../configuration/plugins/hermes-client/) plugin
    - [hermes-client](../configuration/hermes-client/) for generic hermes-client settings, and set up your client datamodel

6. Run the appropriate hermes-client by following the [Run](../run/) section

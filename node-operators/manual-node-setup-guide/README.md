---
description: >-
  This is a step-by-step guide for setting up a Pocket node on hosted hardware.
  After completing the steps outlined here, you’ll have a fully functional
  Pocket node up and running.
---

# Manual Node Setup Guide

## Who is this guide for? <a href="#who-is-this-guide-for" id="who-is-this-guide-for"></a>

This guide is for anyone interested in running Pocket nodes. While the goal is to keep things simple, the assumption is that you have some general blockchain and computer networking knowledge, and some Linux terminal experience. There are other free open source projects that abstract away a large amount of the technical requirements/knowledge and make everything much simpler with a GUI. ###NOTE****_TBA

### Background&#x20;

The main utility of a Pocket node is to relay transactions to other blockchains. So, Pocket nodes need access to other nodes for the blockchains they’ll be relaying to. However, the focus of this guide is just on setting up a Pocket node that will relay information from the Pocket network's own Blockchain, essentially, through itself. **Setting up nodes for other blockchains such as Harmony, Ethereum, Avalanche, Polygon or any of the other** [**supported blockchains**](../../reference/supported-chains.md) **is beyond the scope of this guide.**

### What You’ll Need <a href="#what-youll-need" id="what-youll-need"></a>

In order to complete this guide, you’ll need:

1. A server connected to the Internet.

2. A domain name (or a DDNS).

3. The ability to add DNS records to your domain. (If using a DDNS, must have installed and/or configured the client software on your node so it automatically updates the DNS record with the current external IP of your node (necessary for running a node at home or anywhere with a dynamic IP).

4. 15,100+ POKT (if you want to stake your node).

5. About 2 hours to complete setup, configuration, and test everything you just set up.  
  
    _Note: This is an approximate time for setting up a new Hosted Node, installing prerequisites, installing Pocket Core, configuring Pocket, your network, domain/DDNS, and finally how to begin staking with a new validator. This time estimate does NOT take into account the time it takes to sync your new node, which must be completed before you can  stake your validator. This time varies drastically based on several factors: Is a Snapshot used? if not it may take several weeks or more to sync the full Pocket Network Blockchain, Internet bandwidth+Latency, node hardware specs, type of snapshot download method used, and snapshot type: compression formats,  . This gui_

### Manual Setup Tutorial

This is a step-by-step guide for setting up a Pocket node on hosted hardware.&#x20;

While there are many different ways to set up a node, the focus of this tutorial is on keeping things simple and with the minimum of steps, while still focusing on security and stability.

This guide is broken down into five parts:

* [**Part 1: Server setup**](part-1-server-setup.md)
* [**Part 2: Software installation**](part-2-software-installation.md)
* [**Part 3: Pocket configuration**](part-3-pocket-configuration.md)
* [**Part 4: Proxy configuration**](part-4-proxy-configuration.md)
* [**Part 5: Going live**](part-5-going-live.md)

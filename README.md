# OpenVPN check for Datadog
## Overview

TL;DR: a check to send bandwidth usage and the count of active connections to Datadog.

I once ran an OpenVPN service on my VPS. Its sole purpose was to securely tunnel traffic anytime I needed to connect to unencrypted WiFi (something I found deployed to nearly every single hotel I visited last year). Because I used it only infrequently, I hadthe expectation that there would almost never be any connected users, and when there were, there would never be more than one.

Luckily, OpenVPN provides a handy management interface that can be interrogated for such information. If we can get Datadog to drink from the management interface, we could instrument and alert on these things. This is the output from running `load-stats` on the management interface which we will need to parse:

    SUCCESS: nclients=0,bytesin=0,bytesout=0

Once parsed, we can send the results to Datadog to build graphs and alert anytime a connection is made. The steps below will help you start ingesting these metrics, and assume you already have OpenVPN and Datadog installed and running.

## Step 1: Enabling the OpenVPN Management Interface

Open up `/etc/openvpn/openvpn.conf` and add the following line:

    management localhost 7505

Save and restart OpenVPN. OpenVPN should now be listening on port 7505 on localhost. The port choice is arbitrary according to the author, but localhost is a bit more intentional. You could bind the management interface to 0.0.0.0 instead and let anyone connect to it, but this is a bad idea unless you know what you're doing and why you're doing it.

You can use telnet or netcat to verify that the management interface is up:

    $ telnet localhost 7505
    >INFO:OpenVPN Management Interface Version 1 -- type 'help' for more info

Type `help` to bring up the command list and `quit` to exit when you're done.

## Step 2: Installing the Datadog OpenVPN check

1. Edit `openvpn.yaml` and configure to your liking (the default options are sufficient, apart from the tags)
1. Move `openvpn.yaml` to `/etc/dd-agent/conf.d/openvpn.yaml`
1. Move `openvpn.py` to `/etc/dd-agent/checks.d/openvpn.py`
1. Restart the Datadog agent

    /etc/init.d/datadog-agent restart

## Limitations

There are plenty more metrics available through the OpenVPN Management Interface than are exposed by `load-stats`, but because I was mostly interested in the number of connected clients, that's what this check focuses on. If you'd like to see other metrics collected, feel free to open a pull request.

Regarding other operating systems: I've only tested the check and above steps on Linux. PRs are welcome if you have steps for configuring this and installing it on other OSes (and especially if it's completely broken on other OSes and you have a fix!)

## Contributing

Pull requests are welcome!

## Disclaimer

Software is provided as-is. You accept all responsibility in the event that your infracture becomes sentient and christens itself SkyNet (and other less specific misfortunes arising from downloading and running this software).

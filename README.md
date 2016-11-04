# OpenVPN check for DataDog
## Overview

TL;DR: a check to send bandwidth usage and the count of active connections to DataDog.

I run an OpenVPN service on my VPS. Its sole purpose is to securely tunnel traffic anytime I need to connect to unencrypted WiFi (something I found deployed to nearly every single hotel I visited last year). Because of this purpose, I have the expectation that there will almost never be any connected users, and when there is, there will never be more than one.

Luckily, OpenVPN provides a handy management interface that can be interrogated for such information. This DataDog check does just that by attempting to connect to the management interface, running `load-stats` and then closing the connection again. The output of `load-stats` is formatted as such:

    SUCCESS: nclients=0,bytesin=0,bytesout=0

Once parsed, we can send the results to DataDog to build graphs and alert anytime a connection is made.

## Enabling the OpenVPN Management Interface

Open up `/etc/openvpn/openvpn.conf` and add the following line:

    management localhost 7505

Save and restart OpenVPN. OpenVPN should now be listening on port 7505 on localhost. The port choice is arbitrary according to the author, but localhost is a bit more intentional. You could bind the management interface to 0.0.0.0 instead and let anyone connect to it, but this would be a bad idea outside of a tightly controlled environment.

You can use telnet or netcat to verify that the management interface is up:

    $ telnet localhost 7505
    >INFO:OpenVPN Management Interface Version 1 -- type 'help' for more info

Type `help` to bring up the command list and `quit` to exit when you're done.

## Installing the DataDog OpenVPN check

1. Edit `openvpn.yaml` and configure to your liking
1. Move `openvpn.yaml` to `/etc/dd-agent/conf.d/openvpn.yaml`
1. Move `openvpn.py` to `/etc/dd-agent/checks.d/openvpn.py`
1. Restart the DataDog agent

    /etc/init.d/datadog-agent restart

## Limitations

There are plenty more metrics available through the OpenVPN Management Interface than are exposed by `load-stats`, but because I was mostly interested in the number of connected clients, that's what this check focuses on. If you'd like to see other metrics collected, feel free to make a pull request.

Regarding other operating systems: I've only tested the check and above steps on Linux. PRs are welcome if you have steps for configuring this and installing it on other OSes (and especially if it's completely broken on other OSes and you have a fix!)

## Contributing

Pull requests are welcome!

## Disclaimer

Software is provided as-is. You accept all responsibility in the event that your infracture becomes sentient and christens itself SkyNet (and other less specific misfortunes arising from downloading and running this software).

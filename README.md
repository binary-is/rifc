# RIFC

RIFC (IP-list Firewall Configurer) is a Linux shell script for use with `iptables` to create a chain with a list of IP addresses (or networks) that should be allowed according to that chain. This chain can then be used as a white-list for other chains that govern access to specific services.

In other words, it's so that you can keep a text file with a list of IP or network addresses and mirror that list as a chain as a white-list in a Linux firewall.

## Assumptions

**This script was originally only written for the personal use of the author and is published mostly because hey, why not?** The author will gladly help, but just keep in mind that it wasn't published with wide-spread use in mind.

* The user is assumed to know how to use `iptables`. In particular, the firewall is assumed to appropriately block connections if RIFC hasn't allowed them. On its own, RIFC won't disallow or block anything; only allow.

* The given `iptables` chain is assumed to exist.

* The URL is assumed to be accessible (consequently, that you are able to host a text file at some URL).

* The file at the URL is assumed to contain a newline-separated list of IP or network addresses.

## Usage

RIFC must be run as the root user because it manages the firewall.

    sudo rifc https://example.com/list-of-allowed-ips.txt EXAMPLE_IPS

Typically, you'll want to run it with some sort of scheduler like `cron`, at least if you want the chain (EXAMPLE_IPS) to be updated when the text file is updated.

### Example

Let's say that you have an SSH server (TCP port 22) that you only want accessible to the IP address `123.123.123.123` and the network `234.234.234.234/22` (neither of which make sense but that's beside the point).

For this circumstance, we create a text file containing:

    123.123.123.123
    234.234.234.234/22

We'll need to host this text file at some URL and assume that it is `https://example.com/list-of-networks.txt`.

Then we'll create an `iptables` chain which we'll name `SERVICE_SSH` (but can be called whatever you want) on the firewall:

    sudo iptables -N SERVICE_SSH

We will then tell the `INPUT` chain that we'll want connection to SSH (TCP port 22) to go by this new chain:

    sudo iptables -A INPUT -p tcp --dport 22 -j SERVICE_SSH

And then, finally, you'll run RIFC:

    sudo rifc https://example.com/list-of-networks.txt SERVICE_SSH

The output should be something like:

    Adding IP 123.123.123.123... done
    Adding IP 234.234.234.234/22... done

And then, if you were to delete the first line and re-run RIFC:

    Removing IP 123.123.123.123... done

At that point, the IPs and networks specified in `https://example.com/list-of-networks.txt` will be mirrored in the `iptables` chain `SERVICE_SSH`. Connections from those IPs and networks should be accepted by the firewall from then on.

Hopefully, that makes sense. If you have any problems, don't hesitate just sending the author an email.

## Authors

* Helgi Hrafn Gunnarsson <helgi@binary.is>

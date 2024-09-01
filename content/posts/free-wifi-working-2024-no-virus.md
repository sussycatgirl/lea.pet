---
title: "How to pirate airplane Wifi"
date: 2024-07-18T12:53:07+02:00
draft: false
hidden: false
author: Lea
author_url: https://lea.pet/@lea
---

...Or almost any paid public Wi-Fi network, really.

<!--more-->

> Before we begin, I want to make it abundantly clear that bypassing paywalls like this is **illegal**, especially when you're on a plane. This post is entirely for educational purposes, and I am not in any way endorsing you to go out and try this on any network you don't own or have explicit permission on.
>
> All scenarios and network names detailed in this post are **purely fictional**, and any similarities with networks or services from the real world are a coincidence.
>
> Also, don't brag about committing crimes on an airplane until you're [out of the immediate reach of authoritarian government agents](https://lea.pet/notes/9rg6yfxkdq). Not that I'm speaking from experience, of course. I would never do that. {{< neocat >}}

With the boring disclaimers out of the way, let's have some fun!

## So, what are we exploiting?

Almost any public Wi-Fi network utilizes [Captive Portals](https://en.wikipedia.org/wiki/Captive_portal).
You've almost certainly seen these before if you ever connected to a public access point. \
Most of the time, they're used to make sure users agree to the network's Terms of Service or Privacy Policy.

{{< image "free-wifi-working-2024-no-virus/edeka-captive-portal.png" >}}

While mildly annoying for users, this is a perfectly valid use case. Issues arise when Captive Portals are used to authenticate users. To understand why, let me give you a quick run-down on how these Captive Portals actually function.

## How do Captive Portals work?

> This is a simplified explanation, and [Wikipedia goes more in depth about the different methods and specifics](https://en.wikipedia.org/wiki/Captive_portal). This should be enough to understand how it works on a basic level though.

When your device connects to an unauthenticated network, it tests for internet connectivity by sending a HTTP request to a server that's usually ran by the device or OS vendor, for example `http://captive.apple.com`. This server should return a `200 OK` or `204 No Content` status code, sometimes with a basic success message in the response body.

It's important that this is done over HTTP instead of HTTPS because we actually want the network to be able to intercept the request.

If the network wants to serve a Captive Portal, it will block all outgoing traffic and intercept our connectivity check. Instead of the expected `200` or `204` it will send a `302 Temporary Redirect` (or `511 Network Authentication Required`) and redirect to the URL the Captive Portal is hosted at. \
If this happens, the device knows that user interaction or authentication is required to gain internet access and prompts the user with the page.

Once the user has completed the required actions to gain access (agreeing to ToS, providing contact details, completing a payment, etc.), the captive portal can be closed and the user can access the internet.

### The security issue

You might be asking yourself how the network actually identifies and remembers devices - And that is precisely where the issue is.

The only identifier that the router reliably has access to is the MAC address - an 48-bit identifier unique to every device. Or at least that's how it used to be.

Any modern network card can spoof the MAC address, and this is actually used by phones to prevent cross-network tracking by generating a unique MAC address for every Wi-Fi network. However, we can use this to our advantage.

It just so happens that this address is transmitted unencrypted along with other metadata with every Wi-Fi frame. The actual attack is quite simple: We are going to capture Wi-Fi frames from nearby devices, filter for those connected to the target network, clone their MAC addresses and connect to the network until we find an authenticated device.

The downside of this is that access points really don't like it when two devices with an identical MAC address are connected simultaneously. In my experience, when the second device connects the other one will silently lose connectivity but still thinks it is connected to the network. \
This means that we'll essentially be stealing network access that someone else paid for. However, sometimes paid networks offer free access to customers of certain carriers, usually T-Mobile or Vodafone, and it is generally preferable to target these users when possible for ethical reasons. \
If the network offers an access management page to users (which paid networks usually do), you can use it to check whether the target paid for access or is using one of these free benefits.

## Executing the attack

All right, enough talking, time to have some fun!

You will need:

- **A laptop.** Sorry, the tools you'll need aren't made for phones. No, you can't hack a Wi-Fi network on your fancy iPhone 15. You'll also need to run a sensible operating system on the laptop - Windows isn't useful for anything other than basic web browsing, and especially not for professional pentesting tools. And no, don't even think about using WSL.
- **A compatible network adapter.** Unfortunately, not every Wi-Fi card supports the special mode we'll be using to listen to Wi-Fi frames. If your laptop is somewhat recent, the built in Wi-Fi card is likely sufficient, but if not then you'd have to get an external USB Wi-Fi adapter. You'll find out whether your card is supported once we get to [Enabling monitor mode](#enabling-monitor-mode).
- **A target network.** Again - Doing this without explicit permission from the network operator is **illegal**. And yes, regarding the post's title, even though we won't actually be transmitting anything abnormal, it's probably even more illegal to mess around with your network adapter on an airplane, and you really don't want to get caught doing that. Good luck explaining to the flight attendants what the fancy terminal on your laptop means. I don't know who'd be crazy enough to do this on a plane. {{< neocat >}}

### The required software

The software suite we'll be using is called [aircrack-ng](https://www.aircrack-ng.org/). It's available as `aircrack-ng` in the Arch Linux, Fedora and Debian repositories, and probably comes preinstalled on little Timmy's Kali Linux.

### Enabling monitor mode

Since we need more direct access to the network adapter, we'll need to put it into **monitor mode**. I'll briefly cover how to do this here, but if you're having trouble you should make sure to [read the documentation](https://www.aircrack-ng.org/doku.php?id=airmon-ng).

First of all, you'll need the name of your WLAN interface. You can retrieve it with `ip a`. You'll see an output like this:

```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
4: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff permaddr xx:xx:xx:xx:xx:xx
```

It'll most likely be called `wlan0`, `wlo1` or something like that.

Before enabling monitor mode, it's a good idea to disable all network managers, but in my experience it's *probably* fine if you don't. Still, if you're having issues you should probably do that.

Please note that once monitor mode is enabled, you will lose Wi-Fi connectivity until you disable it.

To enable monitor mode (we'll assume the network interface is called `wlo1` here):

```bash
$ sudo airmon-ng start wlo1
```

If the command succeeded, your network interface will have changed its name, likely to `wlo1mon` or something similar. Make sure to run `ip a` again to check what name to use moving forward.

To disable monitor mode later, you can run:

```bash
$ sudo airmon-ng stop wlo1mon
```

Again, remember to substitute `wlo1mon` with the correct name.

### Capturing Wi-Fi frames

With monitor mode enabled, it's time to proceed with capturing a list of connected devices! Later, we will clone the MAC addresses of located devices until we find one that has paid for Wi-Fi access. You'll likely have to go through a couple unauthenticated devices before you find one that works, especially if the network offers a "free" tier. \
I've seen it before that a free plan is offered where traffic is heavily filtered and speed limited to only allow very basic messaging to go through, and this would mean that there's probably a lot of non-paying (and therefore to us useless) devices connected.

Before we continue, let's assume the following:
- The monitor mode network interface is called `wlo1mon`. If yours is named different, you will have to substitute its name in the following commands.
- The network name is `Unitedwifi.com`. We don't know what band it's on yet, but we'll figure that out in a moment using my preferred method, trial and error.
- The network uses a Captive Portal, where the user has three options:
  - **Free**, highly metered internet access, offering only access to locally hosted infotainment media and certain messenger apps
  - **Paid** internet access where the user gets access to higher speeds for a limited time
  - **Free high-speed** internet access for T-Mobile customers
- It also has a web portal on `https://unitedwifi.com` where one can check on their subscription status and pay for internet access.

As established earlier, we'll target these T-Mobile customers to avoid taking access from someone else for something they explicitly paid for.

To capture a list of connected devices, we will use `airodump-ng`.

The command we'll use is:

```bash
$ sudo airodump-ng --essid "Unitedwifi.com" -a --band X wlo1mon
```

Note the `--band X` flag. The three valid options here are `a`, `b` and `g` - You'll have to try all three until you start seeing results. Note that `a` refers to a 5GHz network, while `b` and `g` both refer to a 2.4GHz network. If all three yield no results, make sure you spelled the `essid` (network name) correctly.

Also make sure to run this in a fairly big terminal so you can read the command output properly. You can also try using the `-w <filename>` and `--output-format csv` options to write the output to a file instead of pretty-printing it to your terminal.

You will see two sections - You can mostly disregard the top section, it simply prints a list of discovered access points. The bottom section shows a list of discovered clients. Here, the `STATION` column is the device's MAC address - We'll need these. the `BSSID` column is the MAC address of the access point it's connected to, but it's irrelevant for us.

*I previously mixed up `STATION` and `BSSID` here, sorry about that <3*

> By the way, if you need to avoid looking suspicious, I recommend running everything in a Visual Studio Code terminal with some project open so that you can quickly close it and have plausible deniability that you're simply programming.
>
> {{< image "free-wifi-working-2024-no-virus/funny.jpg" >}}

All right, got a list of devices and their MAC addresses? Stop the airodump-ng command, take note of the MACs and disable monitor mode. Now it's time to find some victim that paid (or in our example, logged in with their T-Mobile account).

### Using the collected MAC addresses

Now that we have a list of MAC addresses, the next step is to spoof our own MAC address. With monitor mode disabled and your network manager re-enabled, head over to your network settings.

You're most likely running NetworkManager, and if you aren't you definitely know what you're doing. The settings screen should look mostly the same across different desktop environments, and you're looking for a "MAC address" or "Cloned MAC address" field. On GNOME I recommend using `nmtui` over the network settings GUI because the latter is fairly unstable in my experience.

Pick a MAC address from the list you gathered earlier and paste it in the MAC address field, then save and disconnect and reconnect the network. Now you want to check whether the device you've cloned paid for internet access (or in this example, is a T-Mobile user), and if not just discard the MAC address and repeat with a different one until you find a suitable target. \
On our example network we can check this by going to `https://unitedwifi.com`, as mentioned above.

Once you found a suitable victim, you should be connected as them and have access to the internet - Until suddenly you don't. Turns out, access points really don't like it when two identical devices are connected to the network!

### Establishing dominance over the victim's device

So when we connected with the other device's MAC address, we actually silently disconnected them. But after a couple minutes, the victim device seems to notice and automatically reconnects - Which in turn kicks us out. \
The symptoms of this are actually really annoying, because the router will never send a deauthentication frame because it thinks it's just the same device that is just reconnecting for some reason. \
As a result, our computer thinks it's still connected, but any attempts at sending packets are now the networking equivalent of screaming into the void.

Luckily, instead of waiting for the Wi-Fi driver to notice and reconnect automatically, we can help it a little bit. For this I wrote a little bash script that repeatedly pings a host on the local network and automatically reconnects once the pings stop going through. \
Feel free to copy it, and make sure to adjust the `PING` and `NETNAME` variables or tweak the script to work best for you:

```bash
#!/bin/bash

PING=unitedwifi.com # <-- the hostname of a local device to ping. in this example unitedwifi.com resolves to a server in the local network.
NETNAME=Unitedwifi.com # <-- the name of the network we're connected to and want to reconnect. admittedly these might be bad example values.

FAILS=0

while [[ true ]]; do
  if ! ping -c 1 -n -w 1 "$PING" &> /dev/null; then
    FAILS=$(($FAILS + 1))

    if [ "$FAILS" -ge 3 ]; then # Sometimes a couple pings can fail for no actual reason, this prevents those from spamming the console
      echo "$FAILS failed pings"
    fi

    sleep 0.1

    if [ "$FAILS" -ge 10 ]; then
      echo "Connection dropped"
      notify-send -e "Connection dropped, reconnecting"
      nmcli c down "$NETNAME"
      nmcli c up "$NETNAME"
      sleep 2

      echo "Waiting for connection"
      while ! ping -c 1 -n -w 1 "$PING" &> /dev/null; do sleep 1; done
      echo "Reconnected"
      notify-send -e "We're so back"
      FAILS=0
    fi
  elif [ $FAILS -ne 0 ]; then
    if [ "$FAILS" -ge 3 ]; then
      echo "Reconnected"
    fi
    FAILS=0
  fi
done
```

And with that, you should be set! Sit back, relax and enjoy your flight.

If you really wanted to, you could set up a Wi-Fi hotspot on your laptop to allow your phone or other devices to reach the internet, but in my experience doing so prevents the reconnect script from working so you might have to tweak it a bit.

Just one final question...

## Why is it so easy?

You'd think that network providers would implement a more secure authentication mechanism than identifying devices solely based on a publicly available identifier. But if you think about it a bit, you realize that it's simply not worth it to dedicate development resources to that. It simply works well enough™️.

The reality is that for some of us terminally online compsci transfems, performing an attack like this is trivial, but the vast majority of their customers would already get bored after the first paragraph of this post. And of those that have the technical ability, almost no one would actually be crazy enough to do it. It's much cheaper to simply let those crazy ones connect for free, maybe wasting a couple cents or even euros (satellite internet is really expensive!) than it would be to dedicate an entire team to finding a solution that doesn't affect negatively the user experience for those that do pay.

Now it'd be a different story if someone used a captive portal to gate access to a private, internal network - That'd be a huge security issue and you should absolutely report it if you ever find something like that in the wild.

But for public networks, they simply don't care. (If you get caught you're still in big trouble though. So, like, don't get caught <3)

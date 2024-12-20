# Migrating from OPNsense OpenVPN legacy

This is a guide on smoothly transitioning away from your OPNsense OpenVPN server legacy setup without forcing clients to change or update their configuration. This is **not** a guide on how to set up OpenVPN on OPNsense. 

### Introduction

OPNsense is sunsetting the original OpenVPN configuration. If, like me, you are running OpenVPN on OPNSense to provide a tunnel for remote clients, this guide is for you.

### Assumptions

* You are running OPNSense 24.7.9 or higher.
* Old OpenVPN server settings are under VPN → OpenVPN → Servers [legacy].
* You were running a tunnel configured using the `IPv4 Tunnel Network` setting.
* You had the `Redirect Gateway` option set: this forces all traffic on the client to go through the VPN.

### Phase 0: Disable the legacy OpenVPN Server

I suggest you do not delete the legacy server setup but disable it until your new setup is working.

1. Go to VPN → OpenVPN → Servers [legacy].
1. Press the green ▶️ button to **disable** the server; the button turns grey.

### Phase 1: Copy the legacy TLS Static Key

You can skip this phase if you do not have `TLS Authentication` enabled in your legacy OpenVPN server.

1. Go to VPN → OpenVPN → Servers [legacy].
1. Copy the `TLS Shared Key` to the clipboard.
1. Go to VPN → OpenVPN → Instances and select the `Static Keys` tab at the top of the page.
1. Press the orange `+` icon near the right of the page to add a key.
1. Set a `Description`, leave `Mode` to `crypt`.
1. Paste the value you copied to the clipboard to the `Static Key` field.

### Phase 2: Set up an OpenVPN Instance

1. Go to VPN → OpenVPN → Instance and select the `Instances` tab at the top of the page.
1. Press the orange `+` icon near the right of the page to add an OpenVPN instance.
1. Enable `advanced mode` using the toggle near the top left of the page.
1. Accept all defaults unless otherwise mentioned below.
1. Set `Description` to whatever you like, but I initially set it to `OpenVPN - New`.
1. Set `Server (IPv4)` to `192.168.2.0/24` or some other IP range that does not overlap with local OPNsense clients.
1. Set `Certificate` to the same value as the old `Server Certificate` value (under VPN → OpenVPN → Servers [legacy]).
1. Only relevant if you had it set in your legacy server: Under `TLS static key` , choose the key you added in `Phase 1` above when you copy/pasted the TLS Static Key.
1. Under `Options`, set `duplicate-cn` to allow multiple clients with the same certificate to connect simultaneously. You should not set this if you want to force a previously connected client to disconnect if the same client connects again. This would force an at-most-one connection for a user.
1. Under `Push Options`, select both `push block-outside-dns` and `push register-dns`.
1. Under `Redirect Gateway`, select `default`.
1. Press the `Save` button.
1. Press the `Apply` button.

### Phase 3: Change the firewall

1. Go to Firewall → Rules → OpenVPN.
2. If you already had a rule for `10.0.8.0/24` (or something along those lines), duplicate it, edit it, and ensure it matches `192.168.2.0/24` or whatever IP range you configured above. Otherwise, add it. (The rule needs to allow `in` traffic for `192.168.2.0/24` )

### Phase 4: Clean up

Once you have confirmed all works...

1. Remove the legacy server under  VPN → OpenVPN → Servers [legacy].
2. Remove the stale firewall rule under Firewall → Rules → OpenVPN.
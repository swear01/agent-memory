---
title: DVLab ASUS OpenVPN on a LAN-only uplink
scope: projects/dvlab-mis
status: active
updated: 2026-11-11
---

The TUF-AX6000 is in ASUS router mode with DHCP disabled, a LAN uplink and disconnected WAN. The OpenVPN server supports TCP, even though the device acts as the Wi-Fi AP. Set a LAN default route via the perimeter firewall for return traffic. A public TCP 443 virtual server on the Zyxel can map to ASUS TCP 1194; the WAN-to-LAN policy matches the translated internal port. Leave existing IKEv2 UDP services alone.

On stock ASUSWRT 3.0.0.4.388, the form submission must include the selected VPN server unit. Without it, the unit-specific settings are not applied and the daemon can fail with an invalid `server-bridge` directive even when the GUI shows TUN. Verify daemon state and the actual TCP listener before opening the perimeter.

In this LAN-only setup, a successful external TLS handshake and a pushed LAN route did not prove LAN access. The ASUS IPv4 FORWARD chain had a DROP policy and no VPN chain jump. Adding a rule scoped to the VPN TUN interface, VPN pool and lab LAN, plus SNAT to the ASUS LAN IP for those same source and destination subnets, made an external SSH probe to a lab host work. A static route on Zyxel alone did not fix it; it was removed. Test an actual LAN service and confirm the public default route remains unchanged.

The stock generator filtered OpenVPN lifecycle hooks from the generated server config, even when NVRAM retained the custom directives. The test rules were installed in the running firewall only; an ASUS reboot or firewall reload can remove them. Do not describe this as reboot-persistent. For a durable deployment, prefer a supported routed attachment or a maintained internal VPN host, then test again from outside the lab. The temporary ASUS management SSH was disabled after inspection.

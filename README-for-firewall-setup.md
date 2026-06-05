## Table of Contents
* [Overview](#overview)
* [Verification](#verification)
* [Workflow](#workflow)
* [Summary](#summary)

---

## Overview
Configure Zone-Based Firewall– Hardening Linux Systems
- The practice involves configuring firewalld on an Ubuntu system to allow only specified traffic and block the rest. Here's an overview of the 7 steps:

- Install & enable firewalld – Install the package, enable and check the service status via systemctl.
- Assign interfaces to zones – Create a custom "client" zone and assign each NIC to either the public or client zone.
- Allow services on the public zone – Permit HTTP, SSH, and DHCP traffic using firewall-cmd --add-service.
- Set source IP for the public zone – Add 192.168.37.10/24 as a source IP to the public zone.
- Allow FTP on the client zone – Add only FTP service to the client zone.
- Set source IP for the client zone – Add 192.168.30.10/24 as a source IP to the client zone.
- Verify zone settings – Confirm both zones are configured correctly using firewall-cmd with zone details.

----

## Verification

 View ALL zones and their details
sudo firewall-cmd --list-all-zones

 View ONLY public zone (SCREENSHOT REQUIRED ✅)
sudo firewall-cmd --zone=public --list-all

 View ONLY client zone (SCREENSHOT REQUIRED ✅)
sudo firewall-cmd --zone=client --list-all

---

## Workflow

```
Ubuntu-01 (192.168.37.10)
        │
        │ hits NIC1 (192.168.37.11)
        ▼
   Ubuntu-Firewall
   firewalld checks source IP → matches Public Zone
   ✅ Allows: HTTP, SSH, DHCP
   ❌ Blocks: everything else
        │
        │ hits NIC2 (192.168.30.11)
        ▼
Ubuntu-02 (192.168.30.10)
   firewalld checks source IP → matches Client Zone
   ✅ Allows: FTP only
   ❌ Blocks: everything else
```

----

## Summary

Ubuntu-Firewall: Run ALL the firewall-cmd commands (all steps 1–7)

Ubuntu-01: Nothing — its IP (192.168.37.10) is already registered as source in Public Zone

Ubuntu-02: Nothing — its IP (192.168.30.10) is already registered as source in Client Zone

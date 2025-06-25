# FortiTutorials
# HA
**FortiGate Clustering Protocol (FGCP)**  
The FortiGate Clustering Protocol (FGCP) is a proprietary HA solution whereby FortiGates can find other member FortiGates to negotiate and create a cluster.  
- A FortiGate HA cluster consists of at least two FortiGates (members) configured for HA operation.  
- All FortiGates in the cluster must be the _same model_ and have the _same firmware_ installed.  
- Cluster members must also have the same hardware configuration (such as the same number of hard disks).  
- All cluster members share the same configurations except for their host name and priority in the HA settings.

The following are best practices for general cluster operation:
- Ensure that heartbeat communication is present (see HA heartbeat interface).
- Enable the session synchronization option in daily operation (see FGSP basic peer setup).
- Monitor traffic flowing in and out of the interfaces.
## HA - CONFIG
>[!TIP]
>All synchronization activity takes place over the HA heartbeat link using **TCP/UDP 703** packets.
## HA - TSHOOT
View HA history [^1]: 

```
diagnose sys ha history read
```

![diag sys ha history read](https://github.com/FortiGeek/fortigeek.github.io/blob/main/gh-DIAG-ha-01.png)  
  
**Messages worth identifying**  
- _HA state change time_  
- _link status changed_ [0 = **down** / 1 = **up**]  
- _heartbeats from FG100ETK18..95 are lost on all hbdev_  
- _<serial #> is elected as the cluster primary of <#> member_
> [!NOTE] 
> The history is limited to 512 entries and is persistent to reboots; each unit keeps track of its own history of events.  
> It'll override the oldest events first when 512 entries are reached.

View HA status info:

```
get system ha status
```

---
## Config:

### Console term length 0 / no pager
```
config system console
set output standard
end
```
___
### Source NAT with VIPs
[VIP as Source NAT](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-use-a-VIP-s-External-IP-Address-for-Source/ta-p/189947)<br><br>
create inbound policy from external (this can be a DENY if it's not needed)<br>
create outbound policy to external with NAT using outgoing interface<br>
create VIP with ```set nat-source-vip enable```
___
### match-vip enable
When an inbound policy contains an explicit DENY, with VIPs enabled, you must edit that DENY policy to match VIPs
```
set match-vip enable
```
"Enable to match packets that have had their destination addresses changed by a VIP (a.k.a NATing)"
___
### Bandwidth widget / max number of monitored interfaces reached
```
di de en
di de traffic interface
```
Check total count # of interfaces listed (port1, port2, port3, LAN, DMZ, WAN, IPSEC_Tuns etc...) **25** is max #

To remove any unnecessary interfaces: 
```
config sys interface
edit <IPSEC_tun01>
set monitor-bandwidth disable
next
end
```
___
### Disable SIP ALG
[Technical Tip: Disabling VoIP Inspection](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Disabling-VoIP-Inspection/ta-p/194131)<br><br>
Disabling SIP inspection can be done partially <disabling SIP-ALG (Layer7), keeping SIP-helper (Layer4) > or completely <disabling both>.

Disable SIP-ALG (Layer7) per vDOM `(config vdom):` 
```
config system settings
set default-voip-alg-mode kernel-helper-based
end
```
this must be done via global config and affects all vdoms. confirm session-helper entry for sip is still 13 with show/grep:
```
config system session-helper
show | grep -fi sip
```
if sip entry is still 13 perform the following (if not, change the entry # after delete):
```
delete 13
end
```
---
## LLDP:
```
config system global
    set lldp-reception enable
end
```
### To configure LLDP reception per VDOM:
```
config system setting
    set lldp-reception enable
end
```
### To configure LLDP reception per interface:
```
config system interface
    edit <port>
        set lldp-reception enable
    next
end
```
----
## Firmware
### upload firmware to primary image slot:
```
execute restore image tftp FGT_61E-v6.4.11.M-build2030-FORTINET.out 192.168.1.100
```
----
### upload firmware to secondary image slot:
```
execute restore secondary-image tftp FGT_61E-v6.4.11.M-build2030-FORTINET.out 192.168.1.100
```
----
### boot fortigate to primary or secondary image slot:
```
execute set-next-reboot primary/secondary
```
----
### TSHOOT:
to confirm its receiving lldp transmissions
```
diag sniff pack any 'ether proto 0x88cc' 4 | grep -i in
```
## TSHOOT:

### DIAG // IPsec VPN tunnel status
Show tunnel summary:
```
get vpn ipsec tunnel summary
```
if output is  selectors(total,up): 1/0 then your tunnel is down
---
### DIAG // IPsec VPN phase1 up or down
check tunnel for phase1 status:
```
diagnose vpn ike gateway list name <blablah_phx>
```
if output is  selectors(total,up): 1/0 then your tunnel is down
---
### DIAG // FortiSwitch
Show VLANs assigned to ports:
```
diag switch vlan list
```
---
Show mac-address table:
```
diag switch mac-address list
```
---
Show interface stats:
```
diag switch physical-ports list
```
---
Show LLDP info:
```
get switch lldp neighbors-summary
```
---
Show LLDP rx/tx send/received stats:
```
get switch lldp stats
```
---
### DIAG // SSL-VPN
Show users connected with SSL-VPN:
```
get vpn ssl monitor
```
---
### DIAG // Interface
Show WAN IP:
```
diag sys waninfo
```
Show interface counters:
```
fnsysctl ifconfig port1
```
Show interface info/stats:
```
diag hardware deviceinfo nic interface
```
---
### LOGS // SSL-VPN [^2]:
SUCCESS: Log & Report -> Events -> VPN Events -> ACTION:
```
tunnel-up
```
ATTEMPT: Log & Report -> Events -> VPN Events -> ACTION:
```
ssl-new-con
```
FAILURE: Log & Report -> Events -> VPN Events -> ACTION:
```
ssl-login-fail
```
---
### DIAG // Link monitor status
```
diagnose sys link-monitor status
```
### LOGS // Link monitor status
Log & Report -> Events -> System Events -> LOG DESCRIPTION:
```
Link monitor status
```
---
### LOGS // HA device election
Log & Report -> Events -> HA Events -> MESSAGE:
```
Virtual cluster's member state moved
```
---
[^1]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Troubleshooting-unexpected-High-Availability-HA/ta-p/228854  
[^2]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-SSL-VPN-event-logs-when-successfully-connected/ta-p/331206

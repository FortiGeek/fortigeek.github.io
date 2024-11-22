# FortiTutorials
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
### TSHOOT:
to confirm its receiving lldp transmissions
```
diag sniff pack any 'ether proto 0x88cc' 4 | grep -i in
```
---
## TSHOOT:

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
### DIAG // SSL-VPN
Show users connected with SSL-VPN:
```
get vpn ssl monitor
```
---
### DIAG // Interface
Show interface counters:
```
fnsysctl ifconfig port1
```
Show interface info/stats:
```
diag hardware deviceinfo nic interface
```
---
### DIAG // HA
View HA history [^1]:  
```
diagnose sys ha history read
```
![diag sys ha history read](https://github.com/FortiGeek/fortigeek.github.io/blob/main/gh-DIAG-ha-01.png)  
  
**Messages worth identifying**  
+ _HA state change time_  
+ _link status changed_ [0 = **down** / 1 = **up**]  
+ _heartbeats from FG100ETK18..95 are lost on all hbdev_  
+ _<serial #> is elected as the cluster primary of <#> member_
> [!NOTE] 
> The history is limited to 512 entries and is persistent to reboots; each unit keeps track of its own history of events.  
> It'll override the oldest events first when 512 entries are reached.

View HA status info:
```
get system ha status
```
---
### LOGS // SSL-VPN
Log & Report -> Events -> VPN Events -> ACTION:
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

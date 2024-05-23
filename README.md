# FortiTutorials
## Config:
### Console term length 0 / no pager
```
config system console
set output standard
end
```
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
## TSHOOT:
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
### LOGS // Link monitor status
Log & Report -> Events -> System Events:
```
Log Description: Link monitor status
```
---
### LOGS // HA device election
Log & Report -> Events -> HA Events

Message:
```
Virtual cluster's member state moved
```
---
[^1]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Troubleshooting-unexpected-High-Availability-HA/ta-p/228854  

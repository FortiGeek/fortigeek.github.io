# FortiTutorials

### Console term length 0 / no pager
```
config system console
set output standard
end
```

### Bandwidth widget / max number of monitored interfaces reached
```
di de en
di de traffic interface
```
Check total count # of interfaces listed (port1, port2, port3, LAN, DMZ, WAN, IPSEC_Tuns etc...)

To remove any unnecessary interfaces: 
```
config sys interface
edit <IPSEC_tun01>
set monitor-bandwidth disable
next
end
```

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

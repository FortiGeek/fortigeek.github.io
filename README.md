# FortiTutorials

### Console term length 0 / no pager
```
config system console
set output standard
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

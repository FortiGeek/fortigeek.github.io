# FortiTutorials
### Console term length 0 / no pager
### Disable SIP ALG
[Technical Tip: Disabling VoIP Inspection](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Disabling-VoIP-Inspection/ta-p/194131)<br><br>
Disabling SIP inspection can be done partially <disabling SIP-ALG (Layer7), keeping SIP-helper (Layer4) > or completely <disabling both>.
### Disable SIP ALG per vDOM `(vdom0002) #`:
```
config system settings
set default-voip-alg-mode kernel-helper-based
end
``` 
### Console term length 0 / no pager
```
config system console
set output standard
end
```

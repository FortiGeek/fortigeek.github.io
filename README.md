# FortiTutorials
### Console term length 0 / no pager
### Disable SIP ALG
[Technical Tip: Disabling VoIP Inspection](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Disabling-VoIP-Inspection/ta-p/194131)<br><br>
Disabling SIP inspection can be done partially <disabling SIP-ALG (Layer7), keeping SIP-helper (Layer4) > or completely <disabling both>.
- When a firewall policy has a voip-profile applied, SIP-ALG is used over SIP session-helper, even if disabled.
- Disabling SIP session-helper is only necessary if ALL the SIP inspection must be removed.
The commands associated with the SIP-helper will not be relevant if the FortiGate is using SIP-ALG. Fine-tuning SIP-ALG is done through the voip profile.
- Multi-vdom considerations: sip-helper is a global setting. Deleting sip-helper from global context, will make it inaccessible for all VDOMs. SIP-ALG is enabled (by default) and can be disabled per-vdom.
# Disable SIP ALG per vDOM `(vdom0002) #`:
```
config system settings
set default-voip-alg-mode kernel-helper-based
end
``` 

# FortiTutorials
### Disable SIP ALG
Display the SIP session-helper entry number we need to edit (13 in this example)
This is done from _global config_ mode if vDOMs are present:
```
(global) # config system session-helper
(session-helper) # show | grep -fi sip
```
Look for entry number containing sip/udp 5060:
```
config system session-helper
    edit 13
        set name sip <---
        set protocol 17
        set port 5060
    next
end
```
Delete the entry number:
```
delete 13
end
```

Disable SIP ALG
```
config system settings
set default-voip-alg-mode kernel-helper-based
end
```

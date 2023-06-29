# FortiTutorials
#### Disable SIP ALG
Display the SIP session helper and remove it:
```
config system session-helper
show
```
Look for entry containing sip/udp 5060 (entry # may differ):
```
edit 13
set name sip
set protocol 17
set port 5060
next
```
Delete the entry number:
```
delete 13
end
```

2. Disable SIP ALG
```
config system settings
set default-voip-alg-mode kernel-helper-based
end
```
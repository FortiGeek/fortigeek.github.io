

1. When a firewall policy has a voip-profile applied, SIP-ALG is used over SIP session-helper, even if disabled[^1].

### The SIP session helper[^2]
- The SIP session-helper provides basic support for SIP calls passing through the FortiGate by opening SIP and RTP pinholes and by performing NAT of the addresses in SIP messages.
- The SIP session helper:
  - Understands SIP dialog messages.
  - Keeps the states of the SIP transactions between SIP UAs and SIP servers.
  - Translates SIP header and SDP information to account for NAT operations performed by the FortiGate.
  - Opens up and closes dynamic SIP pinholes for SIP signaling traffic.
  - Opens up and closes dynamic RTP and RTSP pinholes for RTP and RTSP media traffic.
  - Provides basic SIP security as an access control device.
  - Uses the intrusion protection (IPS) engine to perform basic SIP protocol checks.
- The SIP session help is set to listen for SIP traffic on TCP or UDP port 5060. 
  - SIP sessions using port 5060 accepted by a security policy that does not include a VoIP profile are processed by the SIP session helper.
- The SIP session helper is disabled by default and must be enabled for the SIP session helper to process VoIP traffic.
  - If the FortiGate is operating with multiple VDOMs, each VDOM can have a different SIP session helper configuration.
  - If you want to use the SIP session helper you need to enter the following commands to disable the SIP ALG and to enable the SIP session helper:
```
config system settings
set default-voip-alg-mode kernel-helper-based
set sip-helper enable
end 
```
 - To use the SIP session helper you must not add a VoIP profile to the security policy. 
   - If you add a VoIP profile, SIP traffic bypasses the SIP session helper and is processed by the SIP ALG.

### The SIP ALG[^3]
- The FortiOS SIP Application Layer Gateway (ALG) allows SIP calls to pass through a FortiGate by opening SIP and RTP pinholes and performing source and destination IP address and port translation for SIP and RTP packets.
- The SIP ALG provides the same basic SIP support as the SIP session helper.
- Additionally, the SIP ALG provides a wide range of features:
  - protect your network from SIP attacks
  - apply rate limiting to SIP sessions
  - checks the syntax of SIP and SDP content of SIP messages
  - provide detailed logging and reporting of SIP activity. 
- By default all SIP traffic is processed by the SIP ALG.
  - If the policy that accepts the SIP traffic includes a VoIP profile, the SIP traffic is processed by that profile.

### [How to disable SIP ALG ](https://community.fortinet.com/t5/Support-Forum/how-to-disable-SIP-ALG/m-p/70822)
As mentioned in the KB, it's the original/old/not-VDOM-based way to handle SIP sessions before they implemented ALG(proxy base). Both basically do the same, and in case you don't want a FW to tweak SIP sessions, you need to disable both.

### [How to confirm if FortiGate is using SIP Session Helper or SIP ALG](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-confirm-if-FortiGate-is-using-SIP-Session/ta-p/190757?externalID=FD38087)
By default, FortiGate is using SIP ALG to process SIP traffic.

### To verify it checking configuration:
`show full system setting | grep default-voip-alg-mode`
```
proxy-based = default. SIP ALG is used.
kernel-helper-based = sessions are probably used (check the config systme settings set sip-helper enable for full session helper mode)
```
[^1]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Disabling-VoIP-Inspection/ta-p/194131
[^2]: https://docs.fortinet.com/document/fortigate/6.0.0/handbook/997743/the-sip-session-helper
[^3]: https://docs.fortinet.com/document/fortigate/6.0.0/handbook/48607/the-sip-alg

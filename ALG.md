#### The SIP session helper
##### [6.0.0 Hankbook](https://docs.fortinet.com/document/fortigate/6.0.0/handbook/997743/the-sip-session-helper)
- The SIP session-helper provides basic support for SIP calls passing through the FortiGate by opening SIP and RTP pinholes and by performing NAT of the addresses in SIP messages.
- The SIP session helper:
  - Understands SIP dialog messages.
  - Keeps the states of the SIP transactions between SIP UAs and SIP servers.
  - Translates SIP header and SDP information to account for NAT operations performed by the FortiGate.
  - Opens up and closes dynamic SIP pinholes for SIP signaling traffic.
  - Opens up and closes dynamic RTP and RTSP pinholes for RTP and RTSP media traffic.
  - Provides basic SIP security as an access control device.
  - Uses the intrusion protection (IPS) engine to perform basic SIP protocol checks.

#### The SIP ALG 
##### [6.0.0 Handbook](https://docs.fortinet.com/document/fortigate/6.0.0/handbook/48607/the-sip-alg)
- The SIP ALG provides the same basic SIP support as the SIP session helper.
- Additionally, the SIP ALG provides a wide range of features:
  - protect your network from SIP attacks
  - apply rate limiting to SIP sessions
  - checks the syntax of SIP and SDP content of SIP messages
  - provide detailed logging and reporting of SIP activity. 
- By default all SIP traffic is processed by the SIP ALG.
  - If the policy that accepts the SIP traffic includes a VoIP profile, the SIP traffic is processed by that profile.
### [how to disable SIP ALG ](https://community.fortinet.com/t5/Support-Forum/how-to-disable-SIP-ALG/m-p/70822)
As mentioned in the KB, it's the original/old/not-VDOM-based way to handle SIP sessions before they implemented ALG(proxy base). Both basically do the same, and in case you don't want a FW to tweak SIP sessions, you need to disable both.


### [How to confirm if FortiGate is using SIP Session Helper or SIP ALG](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-confirm-if-FortiGate-is-using-SIP-Session/ta-p/190757?externalID=FD38087)
By default, FortiGate is using SIP ALG to process SIP traffic.
###### To verify it checking configuration:
`show full system setting | grep default-voip-alg-mode`
```proxy-based = default. SIP ALG is used.```
```kernel-helper-based = SIP session helper is used.```


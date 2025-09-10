# FortiTutorials
# HA
**FortiGate Clustering Protocol (FGCP)**  
The FortiGate Clustering Protocol (FGCP) is a proprietary HA solution whereby FortiGates can find other member FortiGates to negotiate and create a cluster.  
- A FortiGate HA cluster consists of at least two FortiGates (members) configured for HA operation.  
- All FortiGates in the cluster must be the _**same model**_ and have the _**same firmware**_ installed.  
- Cluster members must also have the same hardware configuration (such as the same number of hard disks).  
- All cluster members share the same configurations except for their host name and priority in the HA settings.

### Best Practices:
- Heartbeat interfaces:
  - Configure _two interfaces_ to be heartbeat interfaces to avoid [split-brain](https://docs.fortinet.com/document/fortigate/7.2.4/administration-guide/946059/troubleshoot-an-ha-formation#split-brain) scenarios.
  - It is recommended to isolate the heartbeat devices from the user networks by **connecting the heartbeat devices directly to each other (back-to-back)** or to a **dedicated switch** that is not connected to any network.
  - The heartbeat packets contain sensitive information about the cluster configuration and **may use a considerable amount of network bandwidth**.
 >[!CAUTION]
 >HA heartbeat packets should be _**encrypted and authenticated**_ if the cluster interfaces that send HA heartbeat packets flow through existing data networks.
- Heartbeat Interface Priority:
    - In all cases, the heartbeat interface with the highest priority (the higher the number, the higher the priority) is used for all HA heartbeat communication.
    - If the interface fails or becomes disconnected, then the selected heartbeat interface with the next highest priority handles all HA heartbeat communication.
    - If more than one heartbeat interface has the same priority, the **heartbeat interface with the highest priority that is also highest in the heartbeat _interface list_** is used for all HA heartbeat communication.
- Heartbeat bandwidth requirements:
  - The amount of traffic required for session synchronization depends on the connections per second (CPS) that the cluster is processing, since only new sessions (and session table updates) need to be synchronized.
  -  The majority of the traffic processed by the HA heartbeat interface is session synchronization traffic
      -  Other heartbeat interface traffic required to synchronize IPsec states, IPsec keys, routing tables, configuration changes, and so on is usually negligible.
  - Lower throughput HA heartbeat interfaces may increase failover time if they cannot handle the higher demand during critical HA events.
    - The amount of heartbeat traffic can be reduced by:
      - Turning off <code>session-pickup</code> if it is not needed
      - Enabling<code>session-pickup-delay</code>to reduce the number of sessions that are synchronized
      - Using the<code>session-sync-dev</code>option to move session synchronization traffic off of the heartbeat link
 - Enable<code>session-pickup</code>to ensure minimal loss of user sessions during failovers:
      - https://community.fortinet.com/t5/FortiGate/Technical-Tip-HA-session-failover-session-pickup/ta-p/191165
- Enable the session synchronization option in daily operation (see FGSP basic peer setup).
- Monitor traffic flowing in and out of the interfaces.
## HA - CONFIG
>[!WARNING]
> Connectivity with the FortiGate may be temporarily lost as the HA cluster negotiates and the FGCP changes the MAC addresses of the FortiGate's interfaces
### > Define hostname (if not already set)
```
config system global
    set hostname AZ-EDGE01-PRI
end
```
>[!TIP]
>Changing the host name makes it easier to identify individual cluster units in the cluster operations.
### > Recommended primary Active-Passive example config:
```
config system ha
    set group-id 1
    set group-name "AZ-EDGE-CL01"
    set mode a-p
    set password ENC u+FWOhgbEXj4KUsEgujjv4n/WnI6ekcKVQd3+2re63KYMEHJEPuhuDEFU5B8viDU5+JbMsO8SEOTrrn/qV2pstFPaCry4hcxyLBbU/cR/rW4ewgqu1XfOmkDDvjuKBLIs+0Ft9aSvtYZJ2TEKE6LPcp4306vTY34QKfkaG/3GwsrL+Hc9iWL941Yx+/0PUj0kMDOHA==
    set hbdev "<internal7>" 0 "<internal8>" 0 
    set session-pickup enable
    set priority 255
    set override enable
end
```
>[!TIP]
>Enable <code>set override enable</code> and increase the <code>priority</code> (the highest number 255) of the unit that should always be primary (master).
### > Repeat on the other FortiGate devices joining the cluster, giving each device a unique hostname and lesser device priority
## HA - TSHOOT
>[!NOTE]
>All synchronization activity takes place over the HA heartbeat link using **TCP/UDP 703** packets.
>[!IMPORTANT]
>Ensure the heartbeat lost intervals and thresholds are longer than the possible latency in links.

View HA history [^1]: 
```
diagnose sys ha history read
```

![diag sys ha history read](https://github.com/FortiGeek/fortigeek.github.io/blob/main/gh-DIAG-ha-01.png)  
  
**Messages worth identifying**  
- _HA state change time_  
- _link status changed_ [0 = **down** / 1 = **up**]  
- _heartbeats from FG100ETK18..95 are lost on all hbdev_  
- _<serial #> is elected as the cluster primary of <#> member_
> [!NOTE] 
> The history is limited to 512 entries and is persistent to reboots; each unit keeps track of its own history of events.  
> It'll override the oldest events first when 512 entries are reached.

View HA status info:

```
get system ha status
```

---
## Config:

### Console term length 0 / no pager
```
config system console
set output standard
end
```
___
### Source NAT with VIPs<br><br>
When Central SNAT is disabled, 'extintf' defines which source interface can be used in a firewall policy that references the Virtual IP.<br><br>
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
----
## Firmware
### upload firmware to primary image slot:
```
execute restore image tftp FGT_61E-v6.4.11.M-build2030-FORTINET.out 192.168.1.100
```
----
### upload firmware to secondary image slot:
```
execute restore secondary-image tftp FGT_61E-v6.4.11.M-build2030-FORTINET.out 192.168.1.100
```
----
### boot fortigate to primary or secondary image slot:
```
execute set-next-reboot primary/secondary
```
----
### TSHOOT:
to confirm its receiving lldp transmissions
```
diag sniff pack any 'ether proto 0x88cc' 4 | grep -i in
```
## TSHOOT:

### DIAG // IPsec VPN tunnel status
Show tunnel summary:
```
get vpn ipsec tunnel summary
```
if output is  selectors(total,up): 1/0 then your tunnel is down
---
### DIAG // IPsec VPN phase1 up or down
check tunnel for phase1 status:
```
diagnose vpn ike gateway list name <blablah_phx>
```
if output is  selectors(total,up): 1/0 then your tunnel is down
---
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
Show LLDP info:
```
get switch lldp neighbors-summary
```
---
Show LLDP rx/tx send/received stats:
```
get switch lldp stats
```
---
### DIAG // SSL-VPN
Show users connected with SSL-VPN:
```
get vpn ssl monitor
```
---
### DIAG // Interface
Show WAN IP:
```
diag sys waninfo
```
Show interface counters:
```
fnsysctl ifconfig port1
```
Show interface info/stats:
```
diag hardware deviceinfo nic x1
```
---
### LOGS // SSL-VPN [^2]:
SUCCESS: Log & Report -> Events -> VPN Events -> ACTION:
```
tunnel-up
```
ATTEMPT: Log & Report -> Events -> VPN Events -> ACTION:
```
ssl-new-con
```
FAILURE: Log & Report -> Events -> VPN Events -> ACTION:
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
## TSHOOT // DDoS Anomaly
View DDoS-triggered logs:
```
Log & Report -> Anomaly
```
Check for blocked IPs via CLI (also shows cause):
```
diag user quarantine list all
```
Example output:
```
UNIQUEHOST87-P (VDOM_04) # diag user quarantine list all
src-ip-addr       created                  expires                  cause            
3.85.220.164      Fri Jun 27 11:23:43 2025 Fri Jun 27 11:38:43 2025 DOS              
34.230.56.167     Fri Jun 27 11:35:36 2025 Fri Jun 27 11:50:36 2025 DOS  
```
Check for blocked IPs via GUI:
```
Dashboard > Users & Devices > Quarantine (right-click on banned IP and "Delete" to remove banned IP)
```
---
[^1]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Troubleshooting-unexpected-High-Availability-HA/ta-p/228854  
[^2]: https://community.fortinet.com/t5/FortiGate/Technical-Tip-SSL-VPN-event-logs-when-successfully-connected/ta-p/331206

# wefwec
Windows event forwarding tips and tricks

# Tips for windows 2016 -> servers

```
netsh http delete urlacl url=http://+:5985/wsman/
netsh http add urlacl url=http://+:5985/wsman/ sddl=D:(A;;GX;;;S-1-5-80-569256582-2953403351-2909559716-1301513147-412116970)(A;;GX;;;S-1-5-80-4059739203-877974739-1245631912-527174227-2996563517)
netsh http delete urlacl url=https://+:5986/wsman/
netsh http add urlacl url=https://+:5986/wsman/ sddl=D:(A;;GX;;;S-1-5-80-569256582-2953403351-2909559716-1301513147-412116970)(A;;GX;;;S-1-5-80-4059739203-877974739-1245631912-527174227-2996563517)
```

# Firewall

| protocol | port |
| --- | ---- |
| tcp | 5985 |
| tcp | 5986 |


# Configuration (GPO)
```
Server=http://fqdn:5985/wsman/SubscriptionManager/WEC,Refresh=60
```


# Limits (snips from the documentation)

- 2000 - 4000 clients, with 1 to 2 subscriptions
- ~16 gigs ram
- ~4 cpu
- fast disks
- alot of disk

## Configuration parameters

Three parameters control the frequency of the client connections:

- Refresh= (specified in the configuration URL of the GPO)
- DeliveryMaxLatency (specified in the subscription)
- HeartbeatInterval (specified in the subscription)

# Tips for tunkkaus

## Server
- Confirm the Windows Remote Management listener is configured for port 5985:
  - WinRM e winrm/config/listener
- Check configuration via command line tool
  - wecutil es
  - wecutil gs -?


```
Listener

Address = *

Transport = HTTP

Port = 5985

Hostname

Enabled = true

URLPrefix = wsman

CertificateThumbprint

ListeningOn = 127.0.0.1, 192.168.1.1, ::1, fe80::100:7f:fffe%14, fe80::5efe:192.168.1.1%12, fe80::88f7:5cef:3a9c:8f78%11
```

```
C:\Users\administrator>wecutil es
OIKEASECURITYLOGS
SECURITYLOGS

......

C:\Users\administrator>wecutil gs SECURITYLOGS
Subscription Id: SECURITYLOGS
SubscriptionType: SourceInitiated
Description:
Enabled: true
Uri: http://schemas.microsoft.com/wbem/wsman/1/windows/EventLog
ConfigurationMode: MinLatency
DeliveryMode: Push
DeliveryMaxLatencyTime: 30000
HeartbeatInterval: 3600000
Query: <QueryList><Query Id="0" Path="Microsoft-Windows-Sysmon/Operational"><Select Path="Microsoft-Windows-Sysmon/Operational">*</Select></Query></QueryList>
ReadExistingEvents: true
TransportName: HTTP
ContentFormat: RenderedText
Locale: fi-FI
LogFile: ForwardedEvents
PublisherName: Microsoft-Windows-EventCollector
AllowedIssuerCAList:
AllowedSubjectList:
DeniedSubjectList:
AllowedSourceDomainComputers: O:NSG:BAD:P(A;;GA;;;DC)S:

EventSource[0]:
        Address: wsus19.JVE5DTEST.COM
        Enabled: true

C:\Users\administrator>

```

## Client

- ping server (fqdn)
- test ports 5985, 5986
  - Test-NetConnection <computername> -port 5985 -InformationLevel "detailed"

```
PS C:\Users\administrator> Test-NetConnection 2016wef -port 5985 -InformationLevel "detailed"


ComputerName            : 2016wef
RemoteAddress           : 192.168.190.54
RemotePort              : 5985
NameResolutionResults   : 192.168.190.54
MatchingIPsecRules      :
NetworkIsolationContext : Private Network
InterfaceAlias          : Ethernet
SourceAddress           : 192.168.190.51
NetRoute (NextHop)      : 0.0.0.0
TcpTestSucceeded        : True



PS C:\Users\administrator>
```

# Links for reference

- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/events-not-forwarded-by-windows-server-collector
- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/configure-eventlog-forwarding-performance?source=recommendations
- https://learn.microsoft.com/en-us/archive/blogs/technet/mspfe/setting-up-security-event-log-subscriptions-with-windows-server-20032008
- https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-tcp-ip-communication-guidance
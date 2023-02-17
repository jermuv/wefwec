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

| Event delivery optimization options | Description |
| ----------------------------------- | ----------- |
| Normal | This option ensures reliable delivery of events and doesn't attempt to conserve bandwidth. It's the appropriate choice unless you need tighter control over bandwidth usage or need forwarded events delivered as quickly as possible. It uses pull delivery mode, batches 5 items at a time and sets a batch timeout of 15 minutes. |
| Minimize bandwidth | This option ensures that the use of network bandwidth for event delivery is strictly controlled. It's an appropriate choice if you want to limit the frequency of network connections made to deliver events. It uses push delivery mode and sets a batch timeout of 6 hours. In addition, it uses a heartbeat interval of 6 hours.
| Minimize latency | This option ensures that events are delivered with minimal delay. It's an appropriate choice if you're collecting alerts or critical events. It uses push delivery mode and sets a batch timeout of 30 seconds. |

# Encryption

This is taken from the documentation, https://learn.microsoft.com/en-us/windows/security/threat-protection/use-windows-event-forwarding-to-assist-in-intrusion-detection

```
Are WEF events encrypted? I see an HTTP/HTTPS option!
In a domain setting, the connection used to transmit WEF events is encrypted using Kerberos, by default (with NTLM as a fallback option, which can be disabled by using a GPO). Only the WEF collector can decrypt the connection. Additionally, the connection between WEF client and WEC server is mutually authenticated regardless of authentication type (Kerberos or NTLM.) There are GPO options to force Authentication to use Kerberos Only.

This authentication and encryption is performed regardless if HTTP or HTTPS is selected.

The HTTPS option is available if certificate based authentication is used, in cases where the Kerberos based mutual authentication isn't an option. The SSL certificate and provisioned client certificates are used to provide mutual authentication.
```


# Links for reference

- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/events-not-forwarded-by-windows-server-collector
- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/configure-eventlog-forwarding-performance?source=recommendations
- https://learn.microsoft.com/en-us/archive/blogs/technet/mspfe/setting-up-security-event-log-subscriptions-with-windows-server-20032008
- https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-tcp-ip-communication-guidance
- https://learn.microsoft.com/en-us/windows/security/threat-protection/use-windows-event-forwarding-to-assist-in-intrusion-detection
# wefwec
Windows event forwarding tips and tricks

# tips for windows 2016 -> servers

```
netsh http delete urlacl url=http://+:5985/wsman/
netsh http add urlacl url=http://+:5985/wsman/ sddl=D:(A;;GX;;;S-1-5-80-569256582-2953403351-2909559716-1301513147-412116970)(A;;GX;;;S-1-5-80-4059739203-877974739-1245631912-527174227-2996563517)
netsh http delete urlacl url=https://+:5986/wsman/
netsh http add urlacl url=https://+:5986/wsman/ sddl=D:(A;;GX;;;S-1-5-80-569256582-2953403351-2909559716-1301513147-412116970)(A;;GX;;;S-1-5-80-4059739203-877974739-1245631912-527174227-2996563517)
```


# configuration (GPO)
```
Server=http://fqdn:5985/wsman/SubscriptionManager/WEC,Refresh=60
```


# limits (snips from the documentation)

- 2000 - 4000 clients, with 1 to 2 subscriptions
- ~16 gigs ram
- ~4 cpu
- fast disks
- alot of disk

## configuration parameters

Three parameters control the frequency of the client connections:

- Refresh= (specified in the configuration URL of the GPO)
- DeliveryMaxLatency (specified in the subscription)
- HeartbeatInterval (specified in the subscription)


# Links for reference

- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/events-not-forwarded-by-windows-server-collector
- https://learn.microsoft.com/en-us/troubleshoot/windows-server/admin-development/configure-eventlog-forwarding-performance?source=recommendations
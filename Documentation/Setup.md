# Setup

## Prerequisites

1. Windows Server, preferably 2012R2 or later.

    There are no particular hardware requirements for this server. Keep in mind the hard drive for this server will store the logs that you collect. Additionally, you should size this box with enough memory so that race conditions for writing the logs don't occur. 

2. Windows Domain

    Windows Event Forwarding is easiest deployed in a Domain environment. This guide only covers how to enable Windows Event Forwarding in a Domain.

3. WinRM Service

    All clients that will forward logs must have the WinRM service started. WinRM does not have to be configured for event forwarding to work.

## Enable WinRM

On the collector server, open an Administrative command prompt and run the command `winrm qc` to start and enable the WinRM service. The command will prompt for two responses: Do you want WinRM to start automatically and do you want to create a rule in the firewall for WinRM. Answer yes to both of these prompts.

## Enable Event Collection

Open the event viewer and click on the area labeled **Subscriptions**. You will receive a prompt asking if you would like to start the Event Collector Service and configure it to automatically start on boot. Answer yes. 

This concludes the basic configuraiton of the Windows Event Collector, however, you still must configure clients to check in with the collector.

## GPO Settings

**Computer>Policies>Admin Templates>Windows Components>Event Forwarding>Configure target subscription manager**

This setting should be configured to allow clients to connect to your event collector server.

`Server=http://eventcollectorfqdn:5985/wsman/SubscriptionManager/WEC,Refresh=60`

_Note: Refresh=60 was used above. This setting is most likely far too aggressive for a production environment_.

**Computer>Policies>Admin Templates>Windows Components>Event Log Service>Security> Configure log access**

To get the value that should be used here, use the `wevtutil gl security` command and retrieve the **channelAccess** value on any domain member. 

Keep in mind that this setting will overwrite other attempts to set access controls on the event log, so this can break things.

### Notes

* We had issues using CNAMES. Since event forwarding largely uses Kerberos, SPNs must be created correctly for CNAMES. There are several bugs involving CNAMES and Kerberos implementations in Windows. In our case it was easier to eliminated CNAMES and use sites instead.

**_Resources_**
1. [Monitoring What Matters](https://blogs.technet.microsoft.com/jepayne/2015/11/23/monitoring-what-matters-windows-event-forwarding-for-everyone-even-if-you-already-have-a-siem/)
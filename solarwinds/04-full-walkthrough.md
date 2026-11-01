# SolarWinds Lab Walkthrough: Step by Step

## 1. Problem encountered

The SolarWinds web console could not be accessed using:

- `http://localhost:8787`
- `http://localhost/orion`

This showed that the SolarWinds Orion platform configuration was incomplete or failed during setup. The Orion platform depends on SQL Server, IIS, and the SolarWinds services all being configured correctly.

## 2. Running the SolarWinds Configuration Wizard

The Configuration Wizard connects all the required parts of the Orion platform. It configures:

- The SQL Server database connection
- The Orion database schema
- The IIS web console
- The SolarWinds services
- The NetFlow storage databases

When the wizard was first run, it failed because of SQL Server permission problems.

## 3. Root cause of the failure

The wizard failed with this error:

```
Cannot add 'Administrator' user to database 'SolarWindsOrion'.
```

The Windows Administrator account did not have SQL Server sysadmin permissions. SolarWinds needs sysadmin access to change and configure the Orion database during setup.

## 4. Fixing the SQL permission problem

The Administrator account was added to the SQL Server sysadmin role.

Command used:

```
sqlcmd -S .\SOLARWINDS_ORION -E -C -Q "ALTER SERVER ROLE sysadmin ADD MEMBER [SWLAB\Administrator]"
```

What each part does:

- `-S` picks the SQL Server instance.
- `-E` uses Windows authentication.
- `-C` trusts the SSL certificate.
- `ALTER SERVER ROLE` adds the user to the sysadmin role.

After running this command, the Configuration Wizard was run again.

## 5. Successful Configuration Wizard completion

After the SQL permissions were fixed, the wizard configured:

- The SolarWinds Orion database
- The IIS web console
- The SolarWinds services
- The NetFlow storage database

Confirmation message:

```
Database is set up
Website is set up
All services are up and running
```

## 6. Accessing the SolarWinds web console

After the wizard finished, the web console became accessible. Example address:

`http://swlab:8787`

This confirmed the Orion platform was working.

## 7. Running the Network Sonar Discovery Wizard

SolarWinds needs devices to be discovered before it can monitor them. The Network Sonar Discovery Wizard scans the network and finds devices to monitor, such as:

- Servers
- Routers
- Switches
- Firewalls
- Virtual machines

## 8. Network selection

Instead of scanning a whole subnet or IP range, a single IP address was used:

`172.19.245.103`

**Reason:** Scanning large networks during a lab creates extra traffic and takes a long time. Starting with one node allows controlled discovery and easier troubleshooting.

## 9. Virtualization discovery

This section detects VMware or Hyper-V environments. It was **skipped**, because the lab focused on discovering a single Windows server rather than a virtualization platform.

## 10. Agent configuration

SolarWinds supports agent-based monitoring, but most environments use agentless monitoring. Agentless polling uses:

- **WMI** for Windows servers
- **SNMP** for network devices
- **ICMP** (ping)

Agents were **skipped**, because WMI polling was enough for the Windows server.

## 11. Network device configuration management

This section lets SolarWinds connect to network devices using SSH or Telnet to back up their configurations. It applies to routers, switches, and firewalls. The discovered device was a Windows server, so this step was **skipped**.

## 12. SNMP credentials

SolarWinds automatically included the default SNMP community strings, `public` and `private`. These are commonly used by network devices. The Windows server does not rely on SNMP for monitoring, so these settings were left unchanged.

## 13. Windows credentials

Windows credentials were added so SolarWinds could monitor the server using WMI. Username used: `SWLAB\Administrator`.

This lets SolarWinds collect:

- CPU usage
- Memory utilization
- Disk statistics
- Windows services
- Event logs

## 14. Monitoring settings

- Preferred polling method: **WMI**
- ICMP-only devices were excluded, to avoid importing devices that only respond to ping.
- Monitoring was set to **Manually configure monitoring after discovery**, so the discovered resources can be reviewed before they are imported.

## 15. Discovery settings

The default discovery timeouts and retries were used. These control how aggressively SolarWinds scans the network. For a single-server lab, the defaults are enough.

## 16. Discovery scheduling

- Frequency: **Once**
- Execute immediately: **Yes**

This let SolarWinds scan the server right away.

## 17. Discovery results

SolarWinds successfully discovered:

- **Device:** SWLAB, Windows Server 2022
- **Interface:** Ethernet network adapter
- **Volumes:** RAM, Virtual Memory, Fixed Disk
- **Applications:** Microsoft IIS, Microsoft SQL Server

## 18. Configuration profiles

Hardware Inventory and Software Inventory were turned on. This lets SolarWinds collect system information such as:

- CPU model
- Installed memory
- Installed software
- Operating system details

## 19. Importing the node

The final Import Preview confirmed SolarWinds would monitor:

- Node status
- Network interface
- Disk usage
- Memory utilization
- SQL Server
- IIS

After clicking **Import**, SolarWinds added the node to the Orion database and began polling the server.

## 20. Final result

SolarWinds is now actively monitoring the Windows server. Available metrics include:

- CPU utilization
- Memory usage
- Disk capacity
- Response time
- Packet loss
- Application health

This completes the SolarWinds onboarding process for a new monitored node.

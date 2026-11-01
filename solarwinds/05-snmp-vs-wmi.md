# SNMP vs WMI: Monitoring Summary

## Overview

SNMP (Simple Network Management Protocol) and WMI (Windows Management Instrumentation) are both used by monitoring tools like SolarWinds to collect information from systems and devices on a network. They are usually used for different types of devices and give different levels of detail.

## SNMP (Simple Network Management Protocol)

SNMP is mainly used to monitor network equipment such as routers, switches, firewalls, load balancers, printers, and UPS devices.

Typical metrics collected with SNMP:

- Interface bandwidth use
- Packet errors and discards
- Interface status (up or down)
- Device CPU use
- Temperature and fan status
- Power supply health

SNMP also supports **traps**. A trap is an alert message a device sends automatically to the monitoring system when something happens, such as a network interface going down.

## WMI (Windows Management Instrumentation)

WMI is a Windows management framework used mainly to monitor Windows servers and workstations. It lets a monitoring system pull detailed operating system and application information directly from Windows.

Typical metrics collected with WMI:

- CPU use
- Memory use
- Disk capacity and use
- Running services
- Processes
- Windows event logs

Because WMI is built into Windows, it shows more about operating system performance than SNMP does.

## Key differences

| Feature | SNMP | WMI |
| --- | --- | --- |
| Primary use | Network devices | Windows servers |
| Type | Network monitoring protocol | Windows management framework |
| Data collected | Interfaces, device health metrics | CPU, RAM, disk, services, processes |
| Alerts | Supports traps for real-time alerts | Mostly polling-based queries |

## Simple way to remember

SNMP is for network equipment like routers and switches. WMI is for Windows servers, and it shows deeper operating system details like CPU use, memory use, disk use, and services.

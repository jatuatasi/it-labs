# Server Provisioning Workflow

How a server was provisioned in a large edge network, written up from my own work notes. Tool, bot, and system names are shortened to letters on purpose. The practice lab that goes with this write-up is [here](01-server-provisioning-practice-lab.md).

## Short names used in these notes

| Short name | What it is |
| --- | --- |
| CB | The Slack chat bot used to start a provision |
| U | The inventory system that tracks each server and its status |
| L | A local copy of U that the process reads its settings from |
| ECB | The tool that does the network boot prep stage |
| ECF | The tool that does the automated OS install stage |
| SS | The tool used to read detailed logs |

## 1. Purpose

Server provisioning makes sure security patches and firmware updates are done on a server before it goes back into production. The process depends on three parts working together. If any one of them is down, the provision fails.

## 2. Components

- **Salt master:** the configuration tool that manages the Salt image and pushes settings to servers.
- **Provision server:** runs the overall process.
- **L:** the local copy of U that the process uses to read configuration data.

## 3. Workflow overview

A provision was started from Slack through CB. It begins with preflight checks that confirm the server is healthy and ready. Detailed logs for troubleshooting were read in SS, because CB does not show them.

The provision server then handles remote management access and firmware updates and runs the rest of the process. Once the server is ready, a base Ubuntu image is put on it, and the final setup is done by ECF and Salt.

For a clean provision, the cache directory could be wiped on request:

```
provision server01 wipe=true ticket=TKT0000001
```

## 4. The four stages

### Stage 1: Preflight checks

- **Checks the IP address:** makes sure the server has a valid, reachable address on the network, which shows that connectivity and routing work.
- **Verifies the remote management settings (DRAC):** the Dell Remote Access Controller is used for out-of-band management, like power cycling and remote console access.
- **Confirms DNS:** makes sure the server can look up the internal and external names it needs during provisioning.
- **Checks the status in U:** confirms the server is registered and in the right state to start.

### Stage 2: Network boot prep (ECB)

- **Gets the server's MAC address** from the remote management card or DHCP, so the network can identify the right machine. This is needed for network boot and for assigning an IP address.
- **Prepares the server for PXE boot.** PXE lets a server boot from the network instead of a local disk, so it can receive its operating system over the network.
- **Prepares for the OS install.** It sets up the kernel, the startup files, and the boot settings for the FAI (Fully Automated Install) process, so the install starts by itself once the server boots from the network.

### Stage 3: OS install (ECF)

- **Installs the operating system.** ECF runs a hands-off install of a base Ubuntu image made for edge servers.
- **Installs and updates drivers** so the network cards, storage controllers, and other hardware all work.
- **Sets the network settings:** hostname, interface names, static IP or DHCP, gateway, and DNS. This connects the server correctly to the edge network.

*For network problems at this stage, I used a separate internal troubleshooting guide.*

### Stage 4: Salt checks

- **Configures the server.** After ECF finishes, the Salt minion (the small client program on the server) contacts the Salt master. Salt then pushes the settings for that server's role (for example cache, web, or proxy).
- **Applies the Salt highstate.** A highstate applies everything declared for the server: package installs, system settings, services that should be running, user accounts, SSH keys, and logging. If this stage passes, the server is ready for production.

If Salt fails, it can usually be retried by running the Salt setup again. That resets the trust between the minion and the master and forces a fresh configuration.

## 5. Troubleshooting and escalation

| Where it failed | What to do |
| --- | --- |
| Preflight | Check the logs in SS. If it is a hardware problem, send it to the data center team. |
| ECB | Reboot the server. If it still fails, send it to the data center team. |
| ECF | Check for fiber, link, and network card problems. If it cannot be fixed locally, send it to the data center team. |
| Salt setup | Run the Salt setup again. If it still fails, send it to the systems team. |

## 6. Final steps

1. After Salt applies the final settings, the server gets a health check.
2. If every step passes, the server counts as provisioned and goes back into production.

## 7. Additional notes

- **Wipe or no wipe.** The cache directory can be kept or wiped, depending on what is needed. Wiping gives a clean start for the new configuration.
- **Logs.** For detailed investigation, use the logs in SS. CB does not show them.
- **Firmware and remote management updates.** The provision should also update the firmware, the BIOS, the disks, and the lifecycle controller health checks.

## 8. The flow at a glance

| Stage | If it passes | If it fails |
| --- | --- | --- |
| 1. Preflight | Go to ECB | Check the logs in SS |
| 2. ECB | Go to ECF | Reboot the server, then send it to the data center team if it is not fixed |
| 3. ECF | Go to Salt setup | Check fiber, link, and network card, then send it to the data center team if it is not fixed |
| 4. Salt | Server is provisioned and returns to production | Run Salt setup again, then send it to the systems team if it is not fixed |

# Windows Server 2022 VM for SolarWinds

> Screenshots for this lab were kept in Google Drive and are not included here.

## Step 1: Download the Windows Server ISO

Go to Microsoft's evaluation page for Windows Server 2022:
https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

Choose **ISO Download**. This gives you **180 days free**, which is perfect for labs. The file is about 5 to 6 GB.

## Step 2: Create the SolarWinds VM

Open **Hyper-V Manager**, then click **New**, then **Virtual Machine**. Use these settings:

| Setting | Value |
| --- | --- |
| Name | SolarWinds-Server |
| Generation | Generation 2 |
| Memory | 8192 MB (8 GB) |
| CPU | After creation, set to 4 processors |
| Network | Default Switch |
| Disk | 120 GB |
| Installation source | Install from ISO (choose the Windows Server 2022 ISO you downloaded) |

## Step 3: Install Windows Server

Start the VM. The typical install flow is: Language, then Next, then Install Now.

Choose the edition: **Windows Server 2022 Standard (Desktop Experience)**.

**Error seen:** "Insufficient RAM allocation" when clicking Connect. The VM needs enough memory assigned before it will start.

### Type of installation

Choose **Custom: Install Microsoft Server Operating System only (advanced)**.

Upgrade is only used if a server already has an older version of Windows installed. The VM is a blank disk, so Custom is always the choice here.

The next screen shows the virtual hard disk (around 120 GB). Then:

1. Select the disk.
2. Click **Next**.
3. No partitions are needed. Use the entire disk space.

Windows then copies files and installs.

## After the install

### 1. Network discovery prompt

The question "Do you want to allow your PC to be discoverable on this network?" appears on the right side. Click **Yes**. This allows basic network communication inside the lab.

### 2. Windows Admin Center popup

The popup in the middle only suggests another management tool. Close it (X). It is not needed for this lab.

### 3. Quick sanity check (important)

Before installing SolarWinds, confirm the VM network works.

1. Press **Windows key + R**, type `cmd`, and press Enter.
2. Run:

   ```
   ipconfig
   ```

You should see an IPv4 address that looks like `172.x.x.x` or `192.168.x.x`. That means DHCP from the Hyper-V Default Switch is working.

In this lab the VM pulled a DHCP address:

- IPv4 Address: 172.31.232.138
- Gateway: 172.31.224.1

That means the Default Switch is working, the VM has network connectivity, and SolarWinds will be able to discover and monitor other nodes later.

### 4. Rename the server (recommended)

The server starts with a random name.

1. Open **Server Manager**, then **Local Server**.
2. Click the computer name and rename it to something like `SolarWinds-Server`.
3. Restart.

This keeps the lab organized later.

### 5. What comes next

After the reboot:

1. Install the Chrome or Edge update.
2. Run Windows Update.
3. Install SolarWinds Orion / NPM. That is where the real monitoring lab begins.

## Run Windows Update

Inside the Windows Server VM:

1. Click **Start**, then **Settings**, then **Windows Update** (it may be under Updates & Security). If it is not listed, search for it.
2. Click **Check for updates**.
3. Let it install whatever it finds.

Windows Server usually installs things like .NET updates, security patches, and Hyper-V drivers.

SolarWinds needs .NET components, so this step helps avoid install errors. When updates finish, restart the VM if it asks.

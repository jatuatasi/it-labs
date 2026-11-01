# SolarWinds Lab: Download and Install

> Screenshots for this lab were kept in Google Drive and are not included here.

## Download SolarWinds

Inside the **Windows Server VM**, open **Microsoft Edge** and go to:
https://www.solarwinds.com/network-performance-monitor

Click **Download Free Trial**. You fill out a short form:

- Name
- Email
- Company (you can put anything, like "Lab")

It then downloads `SolarWinds-Orion-Installer.exe`. The download is roughly 2 to 3 GB, so it can take a few minutes.

## Installation type

For a lab, keep **Standard Platform** selected.

**Why:**

- It installs the **Main Polling Engine** (the brain of SolarWinds).
- It installs the **Web Console** (the dashboard you log into).
- It installs or uses **SQL Server / SQL Express** for the database.

That is exactly what you want for a single-VM lab.

The other option, **Scalability Add-ons**, is only used when SolarWinds is already installed and you are adding extra polling engines, web servers, or high-availability nodes in large environments. That is enterprise-level and not needed here.

Click **Next**. The following screens are:

1. License Agreement
2. Installation Options
3. Database setup (SQL Express will likely install automatically)

## Installation options (the database question)

This is a normal step. SolarWinds is basically asking: "Where should I store all the monitoring data?"

SolarWinds requires a SQL database, because every metric, alert, and device status gets stored there.

You will usually see two choices:

1. **Install SQL Server Express** (recommended for labs)
2. **Use an existing SQL Server**

For a VM lab, choose **Install SQL Server Express**.

**Why:**

- It installs SQL automatically.
- It needs no extra configuration.
- It is perfect for learning and small environments.

Real companies usually point SolarWinds to a separate SQL server, but that is not needed here.

### What happens next

SolarWinds will:

1. Install SQL Server Express.
2. Create the SolarWinds Orion database.
3. Connect the polling engine to that database.

After that, the installer moves into **Configuration Wizard** mode.

### What to expect

The installation can take **10 to 30 minutes**, because it installs:

- SQL
- IIS
- SolarWinds services
- The web console

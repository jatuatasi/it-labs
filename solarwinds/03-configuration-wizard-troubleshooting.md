# SolarWinds Configuration Wizard: Failure and Fix

## Short summary

**Problem:** The SolarWinds web console was not accessible. The platform configuration was incomplete.

**Cause:** The Configuration Wizard failed because the Windows Administrator account did not have SQL Server **sysadmin** permissions, so the Orion database could not be configured.

**Fix:** Granted sysadmin rights to the Administrator account using `sqlcmd`, then re-ran the SolarWinds Configuration Wizard. It completed successfully.

## Environment

Lab environment using Windows Server with the SolarWinds Orion platform installed. The Configuration Wizard was run after installation to set up the Orion database, web console, and NetFlow components.

## 1. The original problem

The SolarWinds web console could not be opened. These addresses would not load:

- `http://localhost:8787`
- `http://localhost/orion`

That pointed to one of three things:

- The SolarWinds website (IIS) was not configured.
- The SolarWinds services were not fully configured.
- The Orion database connection was broken.

SolarWinds uses the Configuration Wizard to repair all three. So the reason for launching the wizard was: the web console was not accessible, which usually means the Orion platform configuration did not finish properly.

## 2. What the Configuration Wizard does

The wizard rebuilds the SolarWinds platform configuration. It sets up:

- The Orion SQL database connection
- SolarWinds services
- The IIS website / web console
- The NetFlow database
- The log database

Think of it like this:

- SolarWinds install = the software files
- Configuration Wizard = connects everything together

Without running it successfully, the web console will not work.

## 3. Why the wizard failed the first time

The first run failed near the end of the configuration stage with this error:

```
Database configuration failed:
Can not add "Administrator" user to database "SolarWindsOrion".
```

**Meaning:** SolarWinds tried to grant database permissions, but the Windows Administrator account did not have SQL sysadmin privileges. SQL Server blocked the operation.

## 4. Root cause

The Windows Administrator account needed membership in the SQL Server `sysadmin` server role to change the SolarWinds Orion database.

There was a second problem. When trying to run the fix with `sqlcmd`, the connection first failed because SQL Server ODBC Driver 18 enforces encrypted connections. The error said the certificate chain was not trusted.

## 5. Troubleshooting steps

1. Verified the SQL Server instance used by SolarWinds (`SOLARWINDS_ORION`).
2. Opened SQL Server Configuration Manager and confirmed the SQL services were running.
3. Tried to grant sysadmin permissions using `sqlcmd`.
4. Hit the SSL certificate trust error from ODBC Driver 18.
5. Re-ran the `sqlcmd` command with the `-C` flag to trust the certificate.

## 6. The fix

Command used:

```
sqlcmd -S .\SOLARWINDS_ORION -E -C -Q "ALTER SERVER ROLE sysadmin ADD MEMBER [SWLAB\Administrator]"
```

What it does:

- `-S` picks the SQL Server instance.
- `-E` uses Windows authentication.
- `-C` trusts the server certificate.
- `-Q` runs the SQL query. `ALTER SERVER ROLE ... ADD MEMBER` adds the Windows Administrator account to the sysadmin role.

Now SolarWinds has permission to modify the Orion database.

## 7. After the fix

The Configuration Wizard was run again. This time it successfully:

- Configured the database
- Configured the web console
- Started the SolarWinds services

Confirmation messages:

```
Database is set up
Website is set up
All services are up and running
```

## 8. Final outcome

The web console opens at either of these addresses, and the SolarWinds Orion dashboard loads:

- `http://localhost:8787`
- `http://localhost/orion`

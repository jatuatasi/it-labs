# Server Provisioning Practice Lab (Bash and Python)

A practice lab for learning scripting by building a pretend server provisioning tool. Provisioning means getting a server ready for work: checking it, installing the operating system, and setting it up for its job.

Each task is done twice, first in **Bash**, then in **Python**, so I can see how the two compare.

## 1. About this lab

- **Everything here is made up.** The server names (like `server01`), ticket numbers, and the `servers.csv` file are all practice data. No real server names, IP addresses, passwords, or tokens go into the practice scripts.
- **The goal** is to practice reading input, checking it, running steps in order, handling failures, writing a log file, and connecting a script to Slack.

### Short names used in these notes

The real tool and bot names are shortened to letters so I know what each one means.

| Short name | What it is |
| --- | --- |
| CB | The Slack chat bot I typed provision commands into |
| CF | The command-line version of the same tool, run from a Linux shell |
| U | The inventory system that tracks each server and its status |
| L | A local copy of U that the process reads its settings from |
| ECB | The tool that does the network boot prep stage |
| ECF | The tool that does the automated OS install stage |
| SS | The tool I used to read detailed logs (the chat bot did not show them) |

## 2. The provisioning workflow

A provision runs four stages in order. Each stage has to pass before the next one starts.

| Stage | What it does | If it fails |
| --- | --- | --- |
| 1. Preflight | Checks the IP address, remote management settings, DNS, and that the server is registered in U. | Look at the logs in SS. If it is a hardware problem, send it to the hardware team. |
| 2. Network boot prep (ECB) | Looks up the server's MAC address, sets up network boot (PXE), and prepares the install files. | Reboot the server. If it still fails, send it to the hardware team. |
| 3. OS install (ECF) | Installs the operating system (Ubuntu), the drivers, and the network settings. | Check the fiber, the link light, and the network card. If it still fails, send it to the hardware team. |
| 4. Configuration | Installs the extra packages and settings for the server's job, using a tool like Salt (the "highstate"). | Run the configuration setup again. If it still fails, send it to the systems team. |

If all four pass, the server gets a health check and goes back into production.

### The three parts that work together

- **Salt master:** manages the configuration that gets pushed to the server.
- **Provision server:** runs the overall process, including remote management and firmware updates.
- **L:** the local copy of U that the process reads configuration data from.

## 3. The command format

The practice command copies the style of the commands I typed into CB in Slack. The server name comes first. Every option after it is written as `name=value`.

```
provision server01 ticket=TKT0000001 wipe=true
```

From a Linux shell, CF did the same job with flags instead (for example `-t` for the ticket). The lab scripts copy the CB style.

| Option | What it does | Allowed values |
| --- | --- | --- |
| server | The server to rebuild. Always first, with no `server=` in front. | Any server name in `servers.csv` |
| ticket | The ticket this work belongs to. | `INC` or `TKT`, followed by 7 digits |
| wipe | Erases the cache partition before the rebuild. | `true` or `false`. Default is `false`. |
| os | Which Ubuntu version to install. | `bionic` or `focal` |
| repo | Which software source the server pulls from. | `prod`, `test`, or `edge` |

Only `wipe` has a default. Any other missing option should be treated as unknown and reported.

## 4. The wipe option and safety rules

`wipe=true` erases the cache partition before the rebuild. A wipe cannot be undone, so nothing should be erased unless someone asks for it.

- **Default is no wipe.**
- **Only some server roles have a cache partition.** A script cannot count on people remembering that, so it has to check.

**Rules for the lab script**

| Server role | With `wipe=true` | With no wipe option |
| --- | --- | --- |
| cache | Allowed. Print "cache will be wiped" first. | Run with no wipe. |
| web | Allowed. Print "cache will be wiped" first. | Run with no wipe. |
| proxy | Stop with a clear error. | Run with no wipe. |
| director | Stop with a clear error. | Run with no wipe. |

Two more guards:

- Stop if the server name is not in `servers.csv`.
- Stop if its status in U is `DFR` (down for repair).

## 5. Lab tasks and progress

Tick a box when the script works. When reviewing my code, point out what is wrong and what to look up, and I fix it myself.

### Task 1: Read the command

Pull the server name, ticket, and wipe value out of:

```
provision server01 ticket=TKT0000001 wipe=true
```

Also test it with wipe left off, and with the options in a different order.

- [ ] Task 1 in Bash
- [ ] Task 1 in Python

### Task 2: Check the input

- Is the server in `servers.csv`?
- Does the ticket look right?
- Does the wipe rule above allow it?
- Is the status something other than `DFR`?

- [ ] Task 2 in Bash
- [ ] Task 2 in Python

### Task 3: Run the four stages as fake steps

Preflight, network boot prep, OS install, configuration. Each prints pass or fail, and a failure stops the run.

- [ ] Task 3 in Bash
- [ ] Task 3 in Python

### Task 4: Handle failures the way the workflow says

- Preflight fails: check the logs.
- Network boot prep fails: reboot, then send it to the hardware team.
- OS install fails: check fiber, link, and network card, then send it to the hardware team.
- Configuration fails: run setup again, then send it to the systems team.

Add a few made-up error codes of my own to practice with.

- [ ] Task 4 in Bash
- [ ] Task 4 in Python

### Task 5: Write a log file

One line per run with the time, server, ticket, wipe value, and result.

- [ ] Task 5 in Bash
- [ ] Task 5 in Python

### Task 6: Connect it to Slack

Python first, then a Bash version. The Bash one posts its results to Slack with `curl`, because answering slash commands needs a small web server, which Python handles better. Build `/server`, `/pop`, and `/provision` commands that read `servers.csv`. This comes last, because the Slack part is only a wrapper around the scripts above.

- [ ] Free Slack workspace made for practice
- [ ] Slack app created, secret keys kept out of the code
- [ ] `/server` works
- [ ] `/pop` works
- [ ] `/provision` works

> Never put Slack keys or tokens in the code or in this repo. Keep them in a separate file that stays on my computer.

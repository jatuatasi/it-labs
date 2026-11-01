# Hyper-V Lab: Building a Windows 10 Pro Virtual Machine

This lab removes an old virtual machine (VM) to free disk space, downloads the Windows 10 installer, builds a new VM in Hyper-V, installs Windows 10 Pro, and saves a clean checkpoint. Every step says what to do and why.

**Date completed:** October 22, 2026
**Host computer:** Windows 11 laptop with Hyper-V
**New VM:** LAB-WIN10

## Words used in this lab, in plain English

| Term | What it means |
| --- | --- |
| Host | The real computer (the Windows 11 laptop) that runs the virtual machines. |
| Hyper-V | The Windows feature that lets one computer run other computers inside it. Hyper-V Manager is the window used to control them. |
| Virtual machine (VM) | A pretend computer that runs inside a window on the host. It has its own Windows, its own files, and its own settings. If you break it, the real laptop is not affected. |
| ISO file | One file that holds everything on a Windows install disc. The VM reads it like a virtual disc. |
| VHDX file | The VM's virtual hard drive. It is one big file on the host's real drive. |
| AVHDX file | A second file Hyper-V creates when you take a checkpoint. It holds only the changes made after the checkpoint. |
| Checkpoint | A saved copy of the VM at one moment, so you can return to it later. Explained fully in Part 6. |
| Generation 2 | The newer style of virtual hardware in Hyper-V. Generation 1 is for older operating systems. |
| Dynamic Memory | A setting where Hyper-V gives the VM only the memory it needs at the moment, up to a limit you set. |
| Default Switch | A built-in virtual network connection that lets the VM reach the internet through the host. |
| Enhanced session | A Hyper-V viewing mode that adds screen resizing and copy and paste. It uses Remote Desktop behind the scenes. |

## Part 1: Remove the old Windows 11 VM to free disk space

**Goal:** the laptop did not have room for two Windows VMs, so the old VM (LAB-CLIENT01, Windows 11) was removed.

### Why turning a VM Off was not enough

A VM that is Off uses no memory and no processor time, but its virtual hard drive file still takes up space on the real drive. Removing the VM from the Hyper-V list does not delete that file. Both steps are needed to get the space back.

### Steps

1. Click the VM (LAB-CLIENT01) in the Virtual Machines list.
2. Click **Settings** in the Actions panel on the right, then click **Hard Drive** under SCSI Controller. The "Virtual hard disk" box shows where the file is stored. Write the path down. *Reason: after the VM is deleted, Hyper-V no longer shows where its files are.*
3. Click **Cancel** to close Settings without changing anything.
4. With the VM still selected, click **Delete** in the Actions panel and confirm. *Reason: this removes the VM from the list.*
5. Open File Explorer, click the address bar, paste the folder path below, and press Enter. Click **Continue** if Windows asks for permission.

   ```
   C:\ProgramData\Microsoft\Windows\Virtual Hard Disks
   ```

6. Delete every file that starts with the VM's name (LAB-CLIENT01). The main file ends in `.vhdx`. The checkpoint files end in `.avhdx`. *Reason: checkpoints keep their own files, and those take space too.*
7. Right-click the Recycle Bin and choose **Empty Recycle Bin**. *Reason: deleted files still use space until the Recycle Bin is emptied.*

### What happened in this lab

The VM's files were already gone from the folder when it was checked. Only the Lab-Ubuntu01 disk file (about 6.9 GB) was left, which belongs to the Ubuntu VM. That VM was also removed later to start from scratch, and its file was deleted too.

### Problems seen

- Pasting the full file path (including the file name) into the File Explorer address bar gave "Windows can't find" the file. **Fix:** paste only the folder path and look for the file in the list.
- A Run box command meant to open the folder with the file highlighted failed because the word "explorer" at the start did not paste in. The simpler folder method above avoids this.

## Part 2: Download the Windows 10 installer (ISO)

**Goal:** get the file the new VM will install Windows from.

1. Go to https://www.microsoft.com/software-download/windows10 in a web browser. *Reason: only download Windows from Microsoft. Other sites can give damaged or unsafe copies.*
2. Click **Download tool now** to get the Media Creation Tool. *Reason: when the page is opened from a Windows computer, it offers this tool instead of a direct ISO link.*
3. Open the tool and accept the license terms.
4. Choose **Create installation media (USB flash drive, DVD, or ISO file) for another PC**, then click **Next**. *Reason: the VM is another PC, and "upgrade this PC" would try to change the real laptop.*
5. Leave the settings as English (United States), Windows 10, and 64-bit (x64), then click **Next**. *Reason: 64-bit matches the Generation 2 VM and today's computers.*
6. Choose **ISO file**, not USB flash drive, then click **Next**. *Reason: the VM reads an ISO file directly. A USB drive is for installing on a real computer.*
7. Save the file in Desktop, then the ISO folder, and name it **Windows**. *Reason: keeping all installers in one folder makes them easy to find, and the VM was set to look for Windows.iso there.*
8. Wait until the tool finishes both parts, "Downloading Windows 10" and "Creating Windows 10 media," before closing it.

### Size of the file

The finished ISO was 4,779,200 KB, about 4.8 GB. Both downloads came out to exactly this size, so that is the normal size for this download. A size far below this would mean the download did not finish.

### After the download

The Media Creation Tool is only a helper and can be deleted once the ISO is saved. Empty the Recycle Bin afterward.

## Part 3: Create the new virtual machine

In Hyper-V Manager, click **New** in the Actions panel, then **Virtual Machine**. The New Virtual Machine Wizard opens. The table shows each screen, what was chosen, and why.

| Wizard screen | What was chosen | Why |
| --- | --- | --- |
| Before You Begin | Next | Information screen only. |
| Name and Location | Name: LAB-WIN10. Location left at the default. | A clear name shows which VM is which. The default location is fine because the files go in the Virtual Hard Disks folder from Part 1. |
| Specify Generation | Generation 2 | Generation 2 is the newer virtual hardware and what current Windows installs expect. It starts up the same way modern real computers do. Generation 1 is for older operating systems. |
| Assign Memory | 4096 MB (4 GB), Use Dynamic Memory checked | Windows 10 runs in 2 GB but feels slow. 4 GB is comfortable. Dynamic Memory means the VM only takes the memory it is using, so the host keeps the rest. If the laptop has only 8 GB total, use 2048 MB instead. |
| Configure Networking | Default Switch | This gives the VM internet access through the host. Without a switch the VM has no network. |
| Connect Virtual Hard Disk | Create a virtual hard disk, name LAB-WIN10.vhdx, size 60 GB | See the notes below. |
| Installation Options | Install an operating system from a bootable image file, pointing to Desktop, ISO, Windows.iso | This is the virtual install disc from Part 2. |
| Summary | Finish | Creates the VM. It shows Off in the list until started. |

### About the 60 GB disk size

- The wizard suggests 127 GB. That number is a limit, not the amount used right away.
- The disk is "dynamically expanding": the file starts small and grows only as data is added inside the VM.
- The limit was lowered to 60 GB so the file can never grow larger than that. Windows 10 needs at least 32 GB, so 60 GB leaves room for lessons.
- The file does not shrink by itself when files are deleted inside the VM.
- The Windows Setup screen later showed "Drive 0 Unallocated Space, 60.0 GB", which confirmed the setting.

## Part 4: Install Windows 10 Pro

### Starting the VM for the first time

1. Right-click **LAB-WIN10** in the list and click **Connect**. A window opens with a Start button.
2. Click inside that window so it receives the keyboard, then click **Start**.
3. When the screen says "Press any key to boot from CD or DVD...", press the **Spacebar** within a few seconds. *Reason: this tells the VM to start from the ISO. If it is missed, the VM finds nothing to start from.* **Fix:** click **Action**, then **Reset**, and try again.

### Windows Setup screens

| Screen | What was chosen | Why |
| --- | --- | --- |
| Activate Windows | "I don't have a product key" | A key is not needed for lab practice. Windows works without one. The limits are a watermark on the desktop and some personalization settings being locked. |
| Select the operating system | Windows 10 Pro | Pro is the edition most workplaces use. It has business features such as joining a domain, Group Policy, BitLocker, and being a Remote Desktop target. These come up in IT support work. Home does not have most of them. |
| License terms | Accept | Required to continue. |
| Which type of installation | Custom: Install Windows only (advanced) | Upgrade only works when Windows is already installed on the computer. The VM is empty. Custom is the normal choice for a fresh install, even though it says "advanced". |
| Where do you want to install Windows | Drive 0 Unallocated Space (60 GB), then Next | This is the empty virtual drive. Windows creates the partitions it needs. Do not click Delete, Format, or New. |

After Next, Windows copies files and restarts the VM several times on its own. If "Press any key to boot from CD or DVD" appears again after a restart, **do not press anything**. *Reason: pressing a key would start the installer again instead of the Windows that was just installed.*

### An error that appeared and how it was fixed

The first attempt stopped with: "Windows cannot find the Microsoft Software License Terms. Make sure the installation sources are valid and restart the installation."

No changes were made to the VM. The ISO was deleted, a fresh copy was downloaded to the same folder with the same name, and the install worked on the second try. The exact cause was not confirmed. Using the same file name and folder meant no VM settings had to be changed.

### Setup questions after the install

| Screen | What was chosen | Why |
| --- | --- | --- |
| Choose privacy settings for your device | All switches turned off, then Accept | A practice VM does not need to share location or usage data with Microsoft. Windows still sends a small amount of required data, which cannot be turned off. |
| Let's customize your experience | Skip | It only changes which tips and ads are shown. |
| Use your Android phone from your PC | No, thanks | Not needed. "Remind me later" would show the offer again. |
| Free trial of Microsoft 365 Personal | Decline Microsoft 365 | It requires a payment method and charges $99.99 a year after the free month. |
| Microsoft 365 Basic | Decline | A paid cloud storage plan at $1.99 a month. |
| Let Cortana help you | Not now | Cortana collects location, contacts, and search and typing history. It is not needed here. |

**Rule of thumb on offer screens:** choose Decline, Skip, or Not now. Choosing Continue on a Microsoft 365 screen starts a paid subscription.

## Part 5: Sign in and fix the lock screen

After setup the VM showed a small "Connect to LAB-WIN10" box and then a lock screen (a mountain lake picture) that did not respond to the Spacebar, mouse clicks, or Enter. Action, then Ctrl+Alt+Delete was grayed out.

### Cause and fix

The VM was using Enhanced session, which sometimes does not respond at the lock screen. The fix was to click **View** in the VM window's menu and uncheck **Enhanced session**. The sign-in screen then appeared right away. The account name and PIN created during Windows setup are used to sign in. If the PIN is forgotten, click "I forgot my PIN" on the sign-in screen.

### What changes without Enhanced session

The VM runs in Basic session. The screen size may be fixed, and copy and paste between the laptop and the VM may not work. This is fine for the lessons. It can be turned back on from the View menu later if needed.

## Part 6: Checkpoints

### What a checkpoint is

A checkpoint is a save point for a VM, like saving a video game before a hard level. It records how the VM looks at that moment. If something goes wrong later, you can go back to that moment and everything done after it is undone.

### How it works behind the scenes

1. When a checkpoint is taken, Hyper-V stops changing the VM's original disk file (the .vhdx).
2. From then on, every change goes into a second file (the .avhdx). The original stays exactly as it was.
3. Applying the checkpoint throws away the changes file and returns to the original. That is why it is fast.

### Why it takes disk space

The changes file keeps growing the longer the VM is used after the checkpoint. Several checkpoints mean several of these files. This is also why the .avhdx files had to be deleted in Part 1.

### Why a clean checkpoint is useful for this lab

The checkpoint was taken right after Windows 10 was installed and tested, before any lessons. If a lesson breaks something, applying the checkpoint returns to a fresh Windows 10 in seconds, with no need to rebuild the VM from the ISO.

### What was on screen

- **Automatic Checkpoint:** made by Hyper-V itself, not by the user.
- **Two checkpoints at 11:12:30 PM and 11:12:47 PM:** the Checkpoint button was clicked twice, 17 seconds apart.
- **Now (green arrow):** this is not a saved checkpoint. It marks where the VM is at the moment and cannot be renamed.

### Steps used

1. In Hyper-V Manager (on the laptop, not inside the VM), right-click **LAB-WIN10** and click **Checkpoint**.
2. In the Checkpoints box, right-click the checkpoint just above the green arrow labeled Now and click **Rename**.
3. Type `Clean_Win10_Pro_Base` and press Enter. *Reason: a clear name shows what the checkpoint is for. The default name is only a date and time.*
4. Click each extra checkpoint once, then click **Delete Checkpoint** in the Actions panel, and confirm. *Reason: extra checkpoints use space. Deleting a checkpoint does not undo your work. It only removes that save point.*

### How to go back to the clean state later

1. Right-click `Clean_Win10_Pro_Base` in the Checkpoints box.
2. Click **Apply**.
3. Click **Apply** again. If it offers to take a new checkpoint first, choose Apply to skip it, unless the current state should be saved.

**Warning:** Apply erases everything done in the VM since that checkpoint. Save any files you need elsewhere first.

## Part 7: Check that the VM works

### Windows version

In Settings, System, About, the VM showed:

- Edition: Windows 10 Pro
- System type: 64-bit operating system
- Installed RAM: 4.00 GB (1.88 GB usable)

The lower usable number is normal because of Dynamic Memory. The command prompt showed version 10.0.19045, which is Windows 10 version 22H2, the final Windows 10 release.

### Internet test

In Command Prompt, the command `ping www.google.com` sent 4 messages and received 4 back, with 0% loss and times of 13 to 18 ms. This proves that the Default Switch works and that the VM can reach the internet. It also shows DNS working, because Windows turned the name www.google.com into a number (an IP address).

## Part 8: Using the VM day to day

### Shut down the right way

1. Inside the VM, click **Start**, then the power icon, then **Shut down**.
2. In Hyper-V Manager, the State column should say **Off**.

*Reason: closing the VM window with the X does not turn the VM off. It keeps running in the background and using the laptop's memory.*

### Start it again

1. Right-click **LAB-WIN10** and click **Connect**.
2. Click **Start**.
3. Do not press a key at "Press any key to boot from CD or DVD". Windows is already installed.

## Part 9: Saving space afterward

- `MediaCreationTool_22H2` can be deleted from the ISO folder. It is only a small helper.
- `Windows.iso` is about 4.8 GB. Keep it to rebuild the VM later, or delete it to save space and download it again if needed.
- Extra checkpoints can be deleted. Keep `Clean_Win10_Pro_Base`.
- After deleting anything, empty the Recycle Bin.

### Safety note

Windows 10 no longer gets regular updates from Microsoft. It is fine for lab practice, but do not use this VM for banking or other private accounts.

## Problems and fixes seen in this lab

| Problem | Fix |
| --- | --- |
| "Windows can't find" the file when a full file path was pasted into File Explorer | Paste only the folder path and look for the file in the list. |
| Setup stopped with "Windows cannot find the Microsoft Software License Terms" | Download a fresh ISO to the same folder and file name, then start the install again. |
| VM shows nothing to boot | Click Action, then Reset, and press the Spacebar at "Press any key to boot from CD or DVD". |
| Lock screen does not respond, Ctrl+Alt+Delete grayed out | In the VM window, click View and uncheck Enhanced session. |
| Forgot the sign-in PIN | Click "I forgot my PIN" on the sign-in screen and follow the steps. |
| Disk space did not come back after deleting a VM | Delete the .vhdx and .avhdx files from the Virtual Hard Disks folder and empty the Recycle Bin. |

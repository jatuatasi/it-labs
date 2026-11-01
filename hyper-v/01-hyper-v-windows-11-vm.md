# Windows Hyper-V Lab: Build a Windows 11 Pro VM

> Screenshots for this lab were kept in Google Drive and are not included here. The password and the security-question answers used in the lab were left out on purpose.

## Step 1: Install Hyper-V (Windows 10/11 Pro, Enterprise, or Education only)

Confirmed the host computer runs Windows 11 Pro.

### Quick pre-check (make sure the PC supports it)

1. Press **Ctrl + Shift + Esc** to open Task Manager.
2. Go to **Performance**, then **CPU**.
3. Confirm it says **Virtualization: Enabled**.

If it says Disabled, it has to be turned on in the BIOS/UEFI settings. In this lab it was already Enabled.

### Turn on Hyper-V in Windows

1. Press the **Windows key**.
2. Type **Turn Windows features on or off** and open it.
3. Scroll down and check **Hyper-V**. This also selects Hyper-V Management Tools and Hyper-V Platform.
4. Windows searches for the files it needs, then asks to restart. Click **Restart Now**.

## Step 2: Create a virtual switch (the networking foundation)

Before building any VM, set up networking. A virtual switch is like a network switch that lives inside your PC. No switch means the VM has no network.

In **Hyper-V Manager**:

1. On the right side, click **Virtual Switch Manager**.
2. On the left, select **External**, then click **Create Virtual Switch**.
3. Name it **ExternalSwitch**.
4. Make sure **External network** is selected.
5. In the dropdown, pick the adapter that is giving you internet. Wi-Fi if you are on Wi-Fi, Ethernet if you are plugged in.
6. Leave "Allow management operating system to share this network adapter" checked.
7. Click **OK**.

**Warning you may see:** "Pending changes may disrupt network connectivity."

**Error you may see:** "Error applying Virtual Switch Properties."

### Fix for the error: reset the network adapters (safe method)

1. Press the **Windows key** and type **Network reset**.
2. Click it, then click **Reset now** and **Yes**.
3. The PC restarts by itself in about 5 minutes (or restart it manually).

This resets the Hyper-V network bindings, leftover virtual network cards, and broken switch attempts. It does not delete your files. After the restart the steps above worked.

### Where things stood

- Hyper-V installed
- Virtualization enabled
- External virtual switch created

The VM will have internet access.

## Step 3: Create the first virtual machine

In Hyper-V Manager, click **New**, then **Virtual Machine** (the Virtual Machine Wizard). Click **Next** on "Before You Begin."

| Screen | What was chosen | Why |
| --- | --- | --- |
| Specify Name and Location | Name: **LAB-CLIENT01**. Leave "Store the virtual machine in a different location" unchecked. | A client VM is lighter and easier to start with than a Server VM. |
| Specify Generation | **Generation 2** | The newer virtual hardware. |
| Assign Memory | **4096 MB** (4 GB), with **Use Dynamic Memory** checked | Dynamic Memory gives the VM more RAM when it needs it and takes RAM back when idle, like a real virtualization host. |
| Configure Networking | **ExternalSwitch** | Connects the VM to Wi-Fi, the router, and the internet. "Not Connected" means no network. "Default Switch" uses NAT, which comes up in later labs. |
| Connect Virtual Hard Disk | Size: **60 GB** | The default of 127 GB is more than a client lab needs. |
| Installation Options | **Install an operating system later** | The Windows ISO had not been downloaded yet. It gets attached after the VM is created. |

## Step 4: Attach the Windows ISO

1. Right-click **LAB-CLIENT01**, then click **Settings**.
2. Under **SCSI Controller**, click **DVD Drive**, then **Add**.
3. If you do not have the ISO, download it from https://www.microsoft.com/en-us/software-download/windows11
4. Select **Image file (.iso)**, click **Browse**, and pick the Windows 11 ISO (usually in Downloads).
5. Click **Open**, then **OK**.

## Step 5: Start the VM and install Windows

1. Right-click **LAB-CLIENT01** and click **Connect**. A black console window opens.
2. Click **Start** (the green button).
3. When you see "Press any key to boot from CD or DVD...", click inside the VM window and press any key.

### Error: "No operating system was loaded"

This is common with Generation 2. The VM did not boot from the ISO because the boot order or Secure Boot needs adjusting.

**Fix 1: boot order**

1. Close the VM window.
2. Right-click **LAB-CLIENT01**, then **Settings**.
3. On the left, click **Firmware** and check the boot order.
4. Repeat Step 5.

**Fix 2: turn off Secure Boot**

1. Close the VM window.
2. Right-click **LAB-CLIENT01**, then **Settings**.
3. On the left, click **Security**.
4. Turn off Secure Boot.
5. Repeat Step 5.

For both fixes: if you do not click inside the VM window and press a key fast enough, the "No operating system" error comes back.

### Windows installation

- On the product key screen, choose **I don't have a product key**. For lab work you do not need activation. Windows installs in evaluation (unactivated) mode, which is fine for Hyper-V labs, Active Directory testing, Group Policy practice, and networking labs. The only sign is a small "Activate Windows" watermark.
- Then choose an edition. Windows 11 Pro was selected, but it **failed because TPM was not enabled**.

### Fix: turn on a virtual TPM (vTPM)

Windows 11 needs TPM 2.0. The host has TPM, but the VM does not unless a virtual one is turned on. This is normal in Hyper-V.

1. Close the VM window.
2. Right-click **LAB-CLIENT01**, then **Settings**.
3. On the left, click **Security**.
4. Under "Encryption Support," check **Enable Trusted Platform Module**. This creates a vTPM.
5. Leave "Encrypt state and virtual machine migration traffic" unchecked.

### Disk setup

Do not create partitions by hand.

1. Select **Disk 0 Unallocated Space (60.0 GB)**.
2. Click **Next**.

Windows creates the EFI, MSR, primary, and recovery partitions by itself. That is how UEFI installs work. Files then start copying.

### After the install

- Windows finished installing. The device was named **LABCLIENT01**.
- The VM rebooted and asked for a Microsoft account. A lab VM should use a **local account** instead.

## Setting up a local account (skipping the Microsoft account)

First idea: click **Sign-in options**, then **Offline account** (or "Domain join instead" if the offline option is hidden). Microsoft hides this option to push cloud sign-in.

When it warns about limited experience, choose **Continue with limited setup**, then create a simple local user:

- Username: `labuser`
- Password: something simple, for lab purposes only (left out of these notes)

The offline account option did not appear because the VM had internet.

### Method 1: disconnect the VM from the network

1. Close the setup screen and turn the VM **Off**.
2. Right-click **LAB-CLIENT01**, then **Settings**.
3. Click **Network Adapter**.
4. Change it from **ExternalSwitch** to **Not connected**, then click **OK**.
5. Start the VM again.

Setup should then say "Let's connect you to a network" with a small **I don't have internet** option, followed by **Continue with limited setup**. That forces a local account.

In this lab the option still did not show, because the build installed (23H2/24H2) forces internet by default.

### Method 2: the bypass command

1. On the "Let's connect you to a network" screen, press **Shift + F10** to open a command prompt inside the VM.
2. Type:

   ```
   OOBE\BYPASSNRO
   ```

3. Press Enter. The VM restarts by itself.

Then continue with the local account setup.

- Security questions were added at this step (the answers are left out of these notes).
- Privacy settings: everything switched **Off**.

The Windows 11 Pro VM was installed successfully. Typing `winver` in the Start menu confirmed **Windows 11 Pro, version 25H2**.

## Step 10: Take your first checkpoint (a critical habit)

1. Go back to **Hyper-V Manager**.
2. Right-click **LAB-CLIENT01**.
3. Click **Checkpoint**.

This saves a snapshot of the clean state. If you break something later, you can roll back instantly. That is why virtualization is so useful.

## Next step: reconnect the network

The VM was still disconnected from the bypass step.

1. Right-click **LAB-CLIENT01**, then **Settings**.
2. Click **Network Adapter**.
3. Change it from **Not connected** back to **ExternalSwitch**.
4. Click **OK**.

Inside the VM, open **Settings**, then **Network & Internet**, and confirm it gets an IP address and shows connected.

Next up: run Windows Updates, verify Hyper-V Integration Services, then start the first troubleshooting lab. Fundamentals come before bigger builds like a Domain Controller.

## Lab 1: Snapshot discipline

In Hyper-V Manager, rename the manual checkpoint to something clear:

Right-click the checkpoint, choose **Rename**, and type `Clean_Win11_Pro_Base`.

This builds professional habits.

## Lab 2: Basic VM management skills

Practice these:

1. Shut the VM down properly (from inside Windows).
2. Start it again.
3. Pause it.
4. Resume it.
5. Save it.
6. Turn it off (hard power off).

Know the difference, because it matters in production:

- **Shut Down:** a graceful OS shutdown.
- **Turn Off:** like pulling the power cord.
- **Save:** a hibernate-style saved state.
- **Pause:** freezes CPU activity.

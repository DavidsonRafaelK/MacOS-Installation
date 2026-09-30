# macOS Installation Guide on VMware Workstation

## Table of Contents
1. [Requirements](#requirements)
2. [Installation](#installation)
3. [Troubleshooting](#troubleshooting)

---

## Requirements

To install macOS on VMware Workstation, ensure you have the following:

- **VMware Workstation Pro**: Virtualization software. Free for personal use since May 2024 and **free for personal and commercial use since November 2024** (version 17.5.2 and later).
- **macOS Image (ISO)**: The installation image for macOS Sequoia 15.
- **RAM**: Minimum 8 GB (16 GB recommended).
- **Disk Space**: Minimum 70 GB of free space (200 GB recommended).
- **VMware Unlocker (DrDonk)**: A tool to unlock the macOS option in VMware Workstation.
- **CPU with AVX2 support**: Required for macOS 13 and above.
- **Virtualization enabled in BIOS**: AMD-V (for AMD) or Intel VT-x (for Intel) must be enabled.

> **⚠️ AMD Users**: Hyper-V and Windows Memory Integrity **must be disabled** before proceeding. See [Pre-Installation Notes for AMD](#pre-installation-notes-for-amd-users).

---

## Installation

### Step 1: Download the Requirements

#### 1. VMware Workstation Pro
- Go to the [Broadcom portal](https://access.broadcom.com/).
- Register for a free account if necessary.
- Navigate to: `Dashboard > All Products > Search: VMware Workstation Pro > Download`.
- Accept the terms and conditions, then download the installer.
- No license key is required. Older 17.5.x installers ask for a license type; select **"Personal Use"** if prompted.

#### 2. macOS Sequoia ISO

Apple does not publish macOS as an ISO, so you need to build one or get one from a third party.

**Option A: build the ISO yourself (recommended, needs access to a Mac)**

This uses Apple's own installer, so you know exactly what is in the image. On a Mac, run in Terminal:

```bash
# Download the full Sequoia installer from Apple into /Applications
softwareupdate --list-full-installers
softwareupdate --fetch-full-installer --full-installer-version 15.x.x   # replace 15.x.x with a Sequoia version from the list above

# Create an empty disk image, write the installer to it, and convert it to an ISO
hdiutil create -o /tmp/Sequoia -size 17g -volname Sequoia -layout SPUD -fs HFS+J
hdiutil attach /tmp/Sequoia.dmg -noverify -mountpoint /Volumes/Sequoia
sudo /Applications/Install\ macOS\ Sequoia.app/Contents/Resources/createinstallmedia --volume /Volumes/Sequoia --nointeraction
hdiutil detach "/Volumes/Install macOS Sequoia"
hdiutil convert /tmp/Sequoia.dmg -format UDTO -o ~/Desktop/Sequoia.cdr
mv ~/Desktop/Sequoia.cdr ~/Desktop/Sequoia.iso
```

Copy `Sequoia.iso` to your Windows machine.

**Option B: download a prebuilt ISO (unofficial)**

- Sites such as [Olarila](https://www.olarila.com/) offer prebuilt macOS Sequoia 15 images.
- These are **third-party builds, not from Apple**. Their contents cannot be verified, so only use them in a VM you do not trust with personal data, and scan the download before use.

> **Note**: Apple's macOS license only allows running macOS on Apple-branded hardware. Running it in VMware on a PC is for testing and learning, and you are responsible for complying with the license.

#### 3. VMware Unlocker (DrDonk v4.2.8)
- Visit the [DrDonk Unlocker Releases page](https://github.com/DrDonk/unlocker/releases).
- Download the latest release (`unlocker428.zip` at the time of writing).
- It supports VMware Workstation Pro 16/17/25/26 on Windows and Linux. It also bundles the macOS VMware Tools ISO (`darwin.iso`) used in Step 7.
- The Unlocker does **not** add AMD CPU support. AMD users also need the CPUID lines in Step 5.

> **Note**: The previously referenced `paolo-projects/unlocker` only supports VMware Workstation 11–15 and has been archived. Use the DrDonk version instead.

---

### Step 2: Pre-Installation Notes for AMD Users

If you are on an **AMD processor**, complete these steps before installing VMware:

1. **Disable Hyper-V**: Open PowerShell as Administrator and run:
   ```
   bcdedit /set hypervisorlaunchtype off
   ```
   Restart your computer after running this command.

2. **Disable Windows Memory Integrity**: Go to `Windows Security > Device Security > Core Isolation Details` and turn off Memory Integrity. Restart if prompted.

3. **Enable AMD-V in BIOS**: Reboot and enter your BIOS/UEFI. Find the virtualization setting (usually under CPU or Advanced settings) and enable **AMD-V** or **SVM Mode**.

Skipping these steps on AMD will cause VMware to fail or macOS to get stuck at the Apple logo.

---

### Step 3: Prepare VMware Workstation

#### 1. Install VMware Workstation Pro
- Run the installer and follow the on-screen instructions.
- If the installer asks for a license type, select **Personal Use**.

#### 2. Install VMware Unlocker (DrDonk)
- **Close VMware Workstation completely** before applying the unlocker.
- Extract the downloaded Unlocker zip file.
- Open the `windows` folder inside the extracted directory.
- Right-click `unlock.exe` and select **"Run as administrator"**.
- Wait for the process to complete. You should see a confirmation message when done.
- If your antivirus flags the file, this is a known false positive — the Unlocker is written in Go, which commonly triggers false positives in Windows Defender. You can verify the SHA256 checksum from the GitHub release page.

> **After any VMware update**, you must re-run `unlock.exe` as the update will overwrite the patched files.

---

### Step 4: Create a New Virtual Machine

1. Open VMware Workstation Pro.
2. Select **"Create a New Virtual Machine"**.
3. Choose **"Custom (Advanced)"** configuration.
4. When asked for the installation disc, select **"I will install the operating system later"**.
5. Set the guest operating system:
   - Select **"Apple Mac OS X"**.
   - Choose **macOS 15** as the version (or the closest available option).
6. Name the VM (e.g., `macOS Sequoia`) and choose a storage location.
7. Allocate system resources:
   - **Processors**: **1 processor** with 4 cores (or up to half your total physical cores). Real Macs of this generation have a single CPU socket, so keep the number of processors at 1 and raise the cores per processor instead.
   - **Memory**: At least 8 GB (8192 MB).
8. Configure network type: choose **NAT** or **Bridged** depending on your preference.
9. Set up the virtual disk:
   - Select **"Create a new virtual disk"**.
   - Allocate a maximum disk size of **200 GB**.
   - Choose **"Store virtual disk as a single file"**.
10. Click **Finish**.

---

### Step 5: Configure and Edit the VMX File

This step is critical. Before booting the VM, you must edit the VMX configuration file.

1. After creating the VM, go to **VM Settings** and under the CD/DVD drive, attach the macOS Sequoia ISO.
2. **Close VMware Workstation**.
3. Navigate to the folder where your VM files are stored (e.g., `Documents\Virtual Machines\macOS Sequoia`).
4. Open the `.vmx` file with a text editor such as Notepad or VS Code.

#### For All Users (Intel and AMD):

Find `ethernet0.virtualDev` and set it to `"vmxnet3"`. If `board-id.reflectHost` already exists in the file, set it to `"FALSE"` and leave it out of the block below. Then add the following lines at the **end** of the file:

```plaintext
smc.version = "0"
board-id.reflectHost = "FALSE"
board-id = "Mac-AA95B1DDAB278B95"
hw.model.reflectHost = "FALSE"
hw.model = "iMac19,1"
serialNumber.reflectHost = "FALSE"
serialNumber = "C01234567890"
```

> **Note**: `board-id` and `hw.model` must describe the same Mac. `Mac-AA95B1DDAB278B95` is the board ID of the `iMac19,1` (2019 iMac), which macOS Sequoia supports. `C01234567890` is a placeholder serial number. The VM will boot with it, but Apple services (App Store, iCloud, iMessage) may refuse to sign in. See [Troubleshooting](#troubleshooting) to generate a realistic one.

#### For AMD Processors — Additional Lines:

AMD CPUs require additional CPUID masking to spoof an Intel CPU identity to the macOS kernel. After adding the lines above, also add the following lines at the end of the file. Do **not** repeat the lines from the previous block — each key must appear only once in the `.vmx` file.

```plaintext
cpuid.0.eax = "0000:0000:0000:0000:0000:0000:0000:1011"
cpuid.0.ebx = "0111:0101:0110:1110:0110:0101:0100:0111"
cpuid.0.ecx = "0110:1100:0110:0101:0111:0100:0110:1110"
cpuid.0.edx = "0100:1001:0110:0101:0110:1110:0110:1001"
cpuid.1.eax = "0000:0000:0000:0001:0000:0110:0111:0001"
cpuid.1.ebx = "0000:0010:0000:0001:0000:1000:0000:0000"
cpuid.1.ecx = "1000:0010:1001:1000:0010:0010:0000:0011"
cpuid.1.edx = "0000:0111:1000:1011:1111:1011:1111:1111"
```

> **Why is this needed?** macOS does not officially support AMD hardware. These CPUID values report a generic Intel CPU (`GenuineIntel`, family 6) identity to the macOS kernel so that it can boot normally. Without this, the VM will hang indefinitely at the Apple logo.

5. Save the `.vmx` file and close the text editor.

---

### Step 6: Install macOS Sequoia

1. Reopen VMware Workstation and start your virtual machine.
2. When the installer boots, select your language.
3. From the macOS utilities screen, open **Disk Utility**.
4. Select **VMware Virtual SATA Hard Drive Media** from the sidebar.
5. Click **Erase**, give the disk a name (e.g., `macOS`), and confirm with **Erase**.
6. Close Disk Utility and click **Install macOS Sequoia**.
7. Follow the on-screen instructions. The VM will restart several times — this is normal. The full installation typically takes 30–60 minutes.
8. Complete the initial macOS setup (language, region, Apple ID, etc.).

---

### Step 7: Install VMware Tools

After macOS is installed, installing VMware Tools will enable features such as display resolution adjustment, clipboard sharing, and drag-and-drop.

1. In VMware Workstation, go to **VM > Install VMware Tools**.
2. On Workstation Pro 16/17, the darwin.iso should be recognized automatically.
3. Follow the installer prompts inside macOS.
4. If the display resolution does not adjust automatically, run the following command in the macOS Terminal:
   ```
   sudo /Library/Application\ Support/VMware\ Tools/vmware-resolutionSet 1920 1080
   ```

---

## Troubleshooting

**Stuck at Apple logo (AMD)**: Verify that the CPUID lines were added correctly to the `.vmx` file. Ensure Hyper-V is disabled and AMD-V is enabled in BIOS.

**"VMware Workstation Unrecoverable Error (vcpu 0)"**: The `smc.version = "0"` line is missing or incorrectly formatted. Re-open the `.vmx` file, verify the entry, and try again.

**No internet in macOS**: Confirm that `ethernet0.virtualDev = "vmxnet3"` is set in the `.vmx` file.

**Black screen or display issues**: Install VMware Tools (Step 7) and use the `vmware-resolutionSet` command to force the correct resolution.

**VMware still shows no macOS option after running Unlocker**: Ensure VMware was fully closed before running `unlock.exe`. Re-run the unlocker as Administrator and restart VMware.

**Cannot sign in to the App Store, iCloud or iMessage**: Apple's servers reject the placeholder serial number `C01234567890`. To fix it, give the VM a realistic serial number for the model it pretends to be:
1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and run `GenSMBIOS.bat`.
2. Generate a serial for **iMac19,1** (the same model as `hw.model` in Step 5).
3. Check the serial at [checkcoverage.apple.com](https://checkcoverage.apple.com). It should be reported as **invalid**. If it belongs to a real Mac, generate another one so you do not clash with someone else's device.
4. Shut down the VM, replace the `serialNumber` value in the `.vmx` file, and boot again.

Sign-in can still fail from a VM even with a good serial number, as Apple may block virtual machines.

---

## Credits

- [DrDonk/unlocker](https://github.com/DrDonk/unlocker) — VMware macOS Unlocker v4 (actively maintained)
- [4fr33 on InsanelyMac](https://www.insanelymac.com/forum/topic/350387-install-macos-on-vmware-workstation-16-amd-ryzen/) — AMD CPUID fix reference
- [AMD-OSX Forum](https://forum.amd-osx.com) — AMD VMware community resources

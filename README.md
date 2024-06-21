# MacOS Installation Guide

## Table of Contents
1. [Requirements](#requirements)
2. [Installation](#installation)

## Requirements
To install MacOS on VMware Workstation, ensure you have the following:

- **VMware Workstation**: Virtualization software.
- **MacOS Image (ISO)**: The installation image for MacOS.
- **RAM**: Minimum 8GB.
- **Disk Space**: Minimum 70GB of free space.
- **VMware Unlocker**: A tool to unlock the MacOS option in VMware.

## Installation

### Step 1: Download the Requirements
1. **VMware Workstation**:
   - Go to the [VMware Workstation download page](https://support.broadcom.com/group/ecx/my-dashboard).
   - Register for an account if necessary to download the software.
   - Follow these steps to download:
     ```plaintext
     Dashboard > All Product > Search: VMware > Product Detail > Download > Accept Terms and Conditions > Enter Your Address > Download
     ```

2. **MacOS ISO**:
   - Visit [Olarila](https://www.olarila.com/).
   - Search for the appropriate MacOS version compatible with your hardware.
   - Download the ISO file.

3. **VMware Unlocker**:
   - Visit the [VMware Unlocker GitHub page](https://github.com/paolo-projects/unlocker).
   - Download the latest version of VMware Unlocker.

### Step 2: Prepare VMware Workstation
1. **Install VMware Workstation**:
   - Follow the installation instructions provided on the VMware website.
  
2. **Install VMware Unlocker**:
   - Extract the downloaded VMware Unlocker files.
   - Run the appropriate script for your OS (e.g., `win-install.cmd` for Windows) with administrator privileges.

### Step 3: Create a New Virtual Machine
1. Open VMware Workstation.
2. Select "Create a New Virtual Machine".
3. Choose "Custom (Advanced)" configuration.
4. Set the guest operating system:
   - Select "Apple Mac OS X".
   - Choose the version (e.g., MacOS 14).
5. Allocate system resources:
   - **Processor**: Allocate the number of processors and cores (recommended: 4 processors and 4 cores).
   - **Memory**: Allocate at least 8GB of RAM.
6. Configure network type:
   - Choose either "NAT" or "Bridged" network.
7. Set up the virtual disk:
   - Select "Create a new virtual disk".
   - Allocate a maximum disk size (recommended: 200GB).
   - Choose "Store virtual disk as a single file".
8. Finish the setup.

### Step 4: Install MacOS
1. Start the newly created virtual machine.
2. Mount the MacOS ISO file to the virtual machine’s CD/DVD drive.
3. Close VMware Workstation and navigate to the folder where your virtual machine files are stored (e.g., `Documents > Virtual Machines`).
4. Open the `.vmx` file with a text editor such as Notepad or VSCode.
5. Add the following line at the end of the file:
   ```plaintext
   smc.version = "0"
   ```
   **For AMD Processors, additionally add the following lines:**
   ```plaintext
   hw.model = "iMac20,2"
   board-id = "Mac-AF89B6D9451A490B"
   smc.version = "0"
   cpuid.0.eax = "0000:0000:0000:0000:0000:0000:0000:1011"
   cpuid.0.ebx = "0111:0101:0110:1110:0110:0101:0100:0111"
   cpuid.0.ecx = "0110:1100:0110:0101:0111:0100:0110:1110"
   cpuid.0.edx = "0100:1001:0110:0101:0110:1110:0110:1001"
   cpuid.1.eax = "0000:0000:0000:0001:0000:0110:0111:0001"
   cpuid.1.ebx = "0000:0010:0000:0001:0000:1000:0000:0000"
   cpuid.1.ecx = "1000:0010:1001:1000:0010:0010:0000:0011"
   cpuid.1.edx = "0000:0111:1000:1011:1111:1011:1111:1111"
   ```
6. Save the `.vmx` file and close the text editor.
7. Reopen VMware Workstation and start your virtual machine.

By following these steps, you should be able to successfully install MacOS on VMware Workstation. Ensure all requirements are met and each step is carefully followed for a smooth installation process.

### Credits:
- [Paolo-Project](https://github.com/paolo-projects/unlocker)
- [4fr33 on InsanelyMac](https://www.insanelymac.com/forum/topic/350387-install-macos-on-vmware-workstation-16-amd-ryzen/)
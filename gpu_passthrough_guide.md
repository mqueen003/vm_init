# Single GPU Passthrough Guide for Proxmox

This guide provides a detailed, end-to-end process for passing a single NVIDIA GPU from a Proxmox host to a guest VM. This process is often referred to as "single GPU passthrough" or "headless GPU passthrough."

**Your Setup:**
*   **Host:** Proxmox VE 8.x on a headless server.
*   **CPU:** AMD Ryzen 5950X
*   **GPU:** NVIDIA RTX 3060 (the only GPU in the system)
*   **Host Boot:** UEFI
*   **Guest VM:** Linux Mint, configured for UEFI (OVMF)

---

### **⚠️ CRITICAL WARNING: READ THIS FIRST**

This procedure will make your Proxmox host **headless**. You will lose all video output from the GPU on the host machine itself. The *only* way to access your Proxmox host after this procedure will be via **SSH** or the **Proxmox Web UI** from another machine on your network.

**DO NOT PROCEED UNLESS:**
*   You have a reliable network connection to the Proxmox host.
*   You are comfortable using the command line.
*   You have confirmed that SSH access to the host is working correctly.
*   You have a backup of any critical data on the Proxmox host.

Mistakes in this process can make your host difficult to access and may require a physical keyboard and monitor to fix. **Proceed with caution.**

---

## Section 1: Host (Proxmox) Preparation

These steps configure the Proxmox host to enable IOMMU and isolate the GPU from the host operating system.

### Step 1.1: Enable IOMMU

IOMMU (Input-Output Memory Management Unit) is a hardware feature that allows the host OS to map I/O devices directly to a VM's memory. For AMD, this is called **AMD-Vi**.

1.  **Edit the GRUB Bootloader Configuration:**
    Open the GRUB config file with a text editor:
    ```bash
    nano /etc/default/grub
    ```

2.  **Modify the `GRUB_CMDLINE_LINUX_DEFAULT` line.** You need to add `amd_iommu=on` and `iommu=pt`. The `pt` (passthrough) option can improve performance for passthrough devices.
    ```diff
    - GRUB_CMDLINE_LINUX_DEFAULT="quiet"
    + GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
    ```

3.  **Update GRUB and Reboot:**
    Save the file, then run the following commands to apply the changes and reboot the host.
    ```bash
    update-grub
    reboot
    ```

4.  **Verify IOMMU is Active:**
    After the host reboots, connect via SSH and run this command:
    ```bash
    dmesg | grep -e DMAR -e IOMMU
    ```
    You should see output indicating that IOMMU is enabled. Look for lines like "AMD-Vi: IOMMU performance counters supported" or similar. If you see messages about IOMMU being disabled or not found, double-check your motherboard's BIOS/UEFI settings to ensure virtualization (SVM/AMD-V) and IOMMU (AMD-Vi) are enabled.

### Step 1.2: Load Required VFIO Modules

You need to tell Proxmox to load the VFIO (Virtual Function I/O) modules, which are essential for passthrough.

1.  **Add modules to `/etc/modules`:**
    This ensures the modules are loaded on boot.
    ```bash
    echo "vfio" >> /etc/modules
    echo "vfio_iommu_type1" >> /etc/modules
    echo "vfio_pci" >> /etc/modules
    echo "vfio_virqfd" >> /etc/modules
    ```

### Step 1.3: Isolate the GPU from the Host

This is the most critical part for a single GPU setup. We must prevent the Proxmox host from using the GPU so it's free for the VM.

1.  **Find the GPU's Device IDs:**
    Use `lspci` to find your GPU and its corresponding audio device. We need the four-digit hex codes (e.g., `10de:2504`).
    ```bash
    lspci -nn | grep -i nvidia
    ```
    You will get output similar to this (your IDs may vary):
    ```
    0b:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA106 [GeForce RTX 3060] [10de:2504] (rev a1)
    0b:00.1 Audio device [0403]: NVIDIA Corporation GA106 High Definition Audio Controller [10de:228e] (rev a1)
    ```
    The IDs we need are `10de:2504` and `10de:228e`.

2.  **Tell `vfio-pci` to Claim the GPU:**
    Create a new modprobe file. This tells the system that the `vfio-pci` driver should claim devices with these IDs.
    ```bash
    echo "options vfio-pci ids=10de:2504,10de:228e" > /etc/modprobe.d/vfio.conf
    ```
    **Important:** Replace the IDs with the ones you found in the previous step.

3.  **Blacklist Host Drivers:**
    To ensure the host doesn't touch the GPU, we need to blacklist the default NVIDIA drivers (`nouveau`, `nvidia`).
    ```bash
    echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
    echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
    echo "blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf
    ```

4.  **Update the Initial Ramdisk and Reboot:**
    This applies all the module and driver changes.
    ```bash
    update-initramfs -u
    reboot
    ```
    **This is the point where you will lose video output on the host.** Ensure SSH is working before you reboot.

### Step 1.4: Verify GPU Isolation

After the host reboots, connect via SSH and verify that the `vfio-pci` driver is now managing the GPU.

1.  **Find the GPU's bus address:**
    ```bash
    lspci -nn | grep -i nvidia
    ```
    The address is at the start of the line (e.g., `0b:00.0`).

2.  **Check the driver in use:**
    Run this command, replacing `0b:00.0` with your GPU's address.
    ```bash
    lspci -k -s 0b:00.0
    ```
    The output should show `Kernel driver in use: vfio-pci`. If it does, you have successfully isolated the GPU.

---

## Section 2: VM Configuration

Now, we assign the isolated GPU to your Linux Mint VM.

1.  **Navigate to the VM Hardware Tab:**
    In the Proxmox Web UI, select your Linux Mint VM. Go to the **Hardware** tab.

2.  **Add the PCI Device:**
    Click `Add` -> `PCI Device`.

3.  **Configure the Passthrough:**
    *   **Device:** Select your NVIDIA GPU from the dropdown. It should be clearly identifiable.
    *   **All Functions:** Check this box. This will automatically include the GPU's audio device (`0b:00.1` in our example) and any other functions on the same device.
    *   **ROM-Bar:** Check this box. It is often required for modern GPUs.
    *   **Primary GPU:** **Do not check this yet.** First, boot the VM and install the drivers. You can enable it later if you want the VM's console to appear on the physically connected monitor.

4.  **Confirm and Boot:**
    Click `Add`. Do not start the VM yet.

---

## Section 3: Guest (Linux Mint VM) Setup

The final steps are performed inside the VM itself.

1.  **Start the VM:**
    Start your Linux Mint VM from the Proxmox UI. Access it via the Proxmox console (noVNC).

2.  **Install NVIDIA Drivers:**
    Once booted into the desktop, use the Driver Manager in Linux Mint to find and install the recommended proprietary NVIDIA driver.
    *   Go to `Menu` -> `Administration` -> `Driver Manager`.
    *   Select the recommended NVIDIA driver and click `Apply Changes`.
    *   Reboot the VM when prompted.

3.  **Verify the GPU is Working:**
    After the VM reboots, open a terminal and run:
    ```bash
    nvidia-smi
    ```
    If this command shows your RTX 3060's details, the passthrough was successful! You can also run `lspci -k` inside the VM to see that the `nvidia` driver is now in use.

---

## Section 4: Common Issues & Final Steps

*   **NVIDIA "Code 43" Error:** This is a common issue where the NVIDIA driver detects it's running in a VM and disables itself. The steps above (especially using `All Functions` and `ROM-Bar`) usually prevent this on Linux guests. If you were passing through to a Windows VM, you would need to add extra configuration to hide the fact it's a VM.
*   **Using the Physical Display:** If you want the VM's display to show up on a monitor physically connected to the RTX 3060, you can now go back to the VM's Hardware options, edit the PCI passthrough entry, and check the **Primary GPU** box.
*   **Proxmox Kernel Updates:** When you update Proxmox, the kernel may be upgraded. This can sometimes undo the driver blacklisting. After a kernel update, it's a good idea to re-run `update-initramfs -u` and reboot to ensure the changes stick.

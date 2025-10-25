# SPICE Display Setup for Linux Mint VM on Proxmox

This guide provides the necessary steps to configure and use SPICE for a Linux Mint VM on a Proxmox host, including package installation, Proxmox configuration, and client-side access.

---

## 🧰 Requirements

- Proxmox VE installed and running
- Linux Mint VM created with:
  - Display set to `SPICE (qxl)`
  - Proper disk and ISO installation completed
- SPICE client on your local machine (`remote-viewer` recommended)

---

## 📦 Inside the Mint VM: Install Required Packages

Open a terminal in the Linux Mint VM and run:

```bash
sudo apt update && sudo apt install -y xserver-xspice spice-vdagent
```

These packages enable dynamic resizing, clipboard sync, and input handling.

---

## 🚀 Enable and Start SPICE Agent

```bash
sudo systemctl enable spice-vdagent
sudo systemctl start spice-vdagent
```

Check its status to ensure it's active:

```bash
systemctl status spice-vdagent
```

You should see `active (running)`.

---

## 🔁 Reboot the VM

```bash
sudo reboot
```

---

## 🖥️ Connect Using SPICE

1. In Proxmox, go to your VM’s **Console** tab.
2. Click **SPICE** to download the `.vv` connection file.
3. On your local machine, run:

```bash
remote-viewer /path/to/your_vm.vv
```

---

## ✅ Confirm SPICE Functionality

After connecting via `remote-viewer`, confirm:

- ✅ Dynamic resolution resizing
- ✅ Clipboard sync (host ⇆ guest)
- ✅ Seamless mouse/keyboard input
- ✅ Optional: Audio works if configured

---

## 🔊 (Optional) Enable Audio Support

If you want audio passthrough:

```bash
sudo apt install spice-vdagent spice-client-gtk pulseaudio
```

Then go to **Mint Settings → Sound → Output**, and select the SPICE audio device.

---

## 🧪 Additional Tips

- You **do not** need to pass through the GPU for SPICE to work.
- You **do not** need to install guest additions like with VirtualBox.
- SPICE is ideal for full desktop virtualization (vs. NoVNC browser view).

---

## 🧹 Next Steps

- Take a snapshot of this VM setup for quick rollback
- Consider enabling QEMU Agent for shutdown/restart controls
- When you're ready, revisit GPU passthrough configuration



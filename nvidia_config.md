## NVIDIA setup on Mint VM
**Run the following command to create a persistend xorg config to automatically start a rendering session on a headless VM**

```bash
sudo nvidia-xconfig --allow-empty-initial-configuration --use-display-device=none -virtual=1920x1080
```

**Confirm the renderer is being utilized with:**

```bash
export DISPLAY=:0
export XAUTHORITY=/home/user/.Xauthority
glxinfo | grep "renderer string"
```

- Should now see the Graphics Card listed
- Confirm NVIDIA drivers are being utilized with the CLI tool `nvidia-smi`

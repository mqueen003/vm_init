## Instructions
- Set up WOL protocol for Proxmox Server
    - `wakeonlan` is already available on **mqhomeserver** -- run command from here
- Ensure *wake by PCI-e* is enabled in the BIOS
- Get MAC address by running

```bash
ip link
```

- Then just send the packet to the mac address with

```bash
wakeonlan -i 192.168.40.255 xxxx:xxxx:xxxx:xxxx
```




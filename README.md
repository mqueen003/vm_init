## Outline
- See **Basic Setup** for themes and essential tools
- Update repository and install whatever selections of **packages** you need
- See **llm_cli** for setting up the Gemini or Codex CLI for help, have it automate some of the other setup aspects
- Add to **Tailnet**
- From the machine you want to access from, run `ssh-copy-id -i user@host` and add to 'known_hosts' file
- If needed, get the official Docker engine from **docker_setup**
- For RDP, set up the `SPICE` client with steps from the chatGPT setup guide
- Mount the NFS with the **fstab** file added to /etc/ and the **.smbcredentials** to your ~/.config directory



**TODO**
- Get a full setup guide (after verifying the working process) for PCI-e passthrough to VM's

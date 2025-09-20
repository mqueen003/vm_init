## Ansible
- Install `ansible` and `sshpass` to allow control of worker nodes without entering password

```bash
sudo apt install -y ansible sshpass
```

- Create a dedicated ansible directory

```bash
mkdir ~/ansible/spark-cluster #example
```


- Define inventory with `hosts.ini`

```ini
[master]
localhost ansible_connection=local

[workers]
192.168.40.xxx
192.168.40.xxx

[all:vars]
ansible_user=worker
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

- Copy ssh key to worker nodes with 

```bash
ssh-copy-id worker@192.168.40.xxx
```


- Test the connection

```bash
ansible all -i hosts.ini -m ping
```

- Install a package on the worker nodes

```bash
ansible workers -i hosts.ini -m apt -a "name=htop state=present update_cache=true" -b -K
```

- Copy a file over to the worker nodes

```bash
ansible workers -i hosts.ini -m copy -a "src=/home/master/config_file_example dest=/home/worker/example_file owner=spark group=spark mode=0664"
```

- Enable  passwordless sudo for the master node

```bash
sudo visudo
```

- Add this line at the bottom

```bash
master ALL=(ALL) NOPASSWD:ALL
```


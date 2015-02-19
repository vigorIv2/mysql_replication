# A working MySQL GTID replication group for development/evaluation and testing purposes

With these files you can setup and provision a locally running in VMs,
MySQL server 5.6 master in a separate VM, and three slaves of the same version in VMs

## Specs

The setup contains 4 VMs:

* MySQL master node with 784Mb of RAM and IP address 192.168.222.71
* MySQL slaves node with 784Mb of RAM and IP addresses 192.168.222.[72-74]

Its been tested with Virtualbox 4.3.20, running in Windows XP

Prerequisites :
- VirtualBox 4.3.20 
- Cygwin (It may work without Cygwin, I just never tested it without in Windows)
- Vagrant 
- vagrant-hostmanager (Install as follows, if you don't have one: "vagrant plugin install vagrant-hostmanager")


Clone this repository.

```bash
$ git clone https://github.com/vigorIv2/mysql_replication
```

Provision the VMs

```bash
$ cd mysql_replication
$ vagrant up
```

After all processes finished 

to access your master VM shell via SSH:

```bash
vagrant ssh mysql_vm71
```
to access your slave VM shell via SSH:

```bash
vagrant ssh mysql_vm[72-74]
```

to play with mysqlfailover script:

```bash
mysqlfailover --master=repl:repl123@192.168.222.71 --discover-slaves-login=repl
```



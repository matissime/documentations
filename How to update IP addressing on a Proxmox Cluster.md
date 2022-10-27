## How to update IP addressing on a Proxmox Cluster

When you change something in your network configuration, rather than its the gateway or the IP addressing, you may come under serious troubleshooting on your Proxmox Cluster, so this document will guide you into the necessary steps needed to update your network settings on Proxmox.

> Note : You must follow these steps on every single node of your cluster to let it able to handle the synchronization between each node once.

So firstly, you will need to change the network interface configuration of the system; it is preferable for you to set a static IP:

```bash
/etc/network/interfaces
```

```bash
auto lo
iface lo inet loopback

iface eth0 inet dhcp

auto vmbr0
iface vmbr0 inet static
        address 10.27.0.201/24
        gateway 10.27.0.1
        bridge-ports eth0
        bridge-stp off
        bridge-fd 0
```



Also, you will need to update your host file and, I also recommend you to include all your node hostname with their corresponding IP according to this example:

```bash
/etc/hosts
```

```bash
127.0.0.1 localhost.localdomain localhost
10.27.0.201 pve1.localhost pve1
10.27.0.202 pve2.localhost pve2
```



Once you have changed the network configuration on the system side, you will need to change it on the PVE side, so we will edit the corosync configuration file that set the network connectivity for the cluster, the part that we will need to change in our file is the nodelist, you will need to put the proper node hostname (name) and IP (ring0_addr) for each entry:

```bash
/etc/pve/corosync.conf
```

```bash
nodelist {
  node {
    name: pve1
    nodeid: 1
    quorum_votes: 1
    ring0_addr: 10.27.0.201
  }
  node {
    name: pve2
    nodeid: 2
    quorum_votes: 1
    ring0_addr: 10.27.0.202
  }
}
```

If you have permissions issues when trying to write the file, which can happen sometimes, you may need to do additional step to put the file the local pve environment to apply your modifications. You can follow these steps at this section: [Corosync Permissions Issues](#Corosync Permissions Issues)



After all the necessary configuration adjustments done, you will need to regenerate the certificate and authentication keys for the SSH cluster communication and the web interfaces, so to do that you will need to clear the ssh known_host of the pve cluster and regenerate certificates:

First, you need to stop the running service related to the PVE clustering:

```
systemctl stop pve-cluster pveproxy corosync
```

Now, you can clear the content of known_host (it is possible that it synchronizes the file on other nodes, so if you see that the file is already empty on others node, for me it was the case, so don't worry):

```bash
echo > /etc/pve/priv/known_host
```

Delete the SSL certificates for the web administration interface (also, it is possible that i will synchronize on other nodes):

```bash
rm /etc/pve/pve-root-ca.pem
rm /etc/pve/priv/pve-root-ca.key
rm /etc/pve/nodes/pve1/pve-ssl.pem
rm /etc/pve/nodes/pve2/pve-ssl.pem
rm /etc/pve/authkey.pub
rm /etc/pve/priv/authkey.key
rm /etc/pve/priv/authorized_keys
```

And finally, you will need to regenerate the certificate and SSH authentication keys to reestablish the connectivity with the new network configuration taken into account:

```bash
pvecm updatecerts --force
```



NB : For me, i run into additionals errors that was really strange, i was able to gain the connectivity between each nodes and make the cluster work again with the new network settings, but after each reboot, i ran into quorum initialization fails, even if i wait a few minutes between each node reboot, the quorum wasn't initializing it-self, but I was able to let it initialize by simply restarting the pve-cluster service on each nodes, which was quite strange and annoying because on each reboot it was required for me to do workaround in order to work again. And I didn't find any causes of this error, so i simply reinstalled all my nodes with a fresh and clean install of PVE as the server wasn't in a live environment and no important VM / CT was on it.



### Troubleshooting :

#### Corosync Permissions Issues

Start your pve filesystem into local mode on all nodes:

```bash
systemctl stop corosync pve-cluster
```

```bash
pmxcfs -l
```

Edit your corosync on a newly created file:

```bash
nano correct_corosync.conf
```

And pull your new corosync configuration over your previous one:

```bash
cp correct_corosync.conf /etc/pve/corosync.conf
```

```bash
cp correct_corosync.conf /etc/corosync/corosync.conf
```

Exit the local mode pve filesystem:

```bash
killall pmxcfs
```

Restart the services:

```bash
systemctl start pve-cluster corosync
```



------

*References :*

*[ **codingpackets.com** ] : https://codingpackets.com/blog/proxmox-certificate-error-fix-after-node-replacement/ "Proxmox Certficate Error Fix After Node Replacement"*

*[ **austienerdythings.com** ] : https://austinsnerdythings.com/2022/05/26/proxmox-corosync-issues/ "Proxmox corosync issues"*
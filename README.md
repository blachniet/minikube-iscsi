# Minikube iSCSI Example

This repository demonstrates usage of iSCSI volumes in Kubernetes. We use
Vagrant to provision two CentOS VMs. The `target` VM simulates iSCSI target
hardware. The `initiator` VM simulates a Kubernetes node. We use Minikube
rather than a full Kubernetes install.

## Set up an iSCSI target

This VM has:
- iSCSI Target

```sh
cd target
vagrant up
vagrant ssh
```

In the VM:

```sh
$ targetcli
> cd backstores/fileio
> create disk01 /var/lib/iscsi_disks/disk01.img 1G
> cd /iscsi
> create iqn.2020-01.com.blachniet:target01
> cd iqn.2020-01.com.blachniet:target01/tpg1/luns
> create /backstores/fileio/disk01
> cd ../acls
> create iqn.2020-01.com.blachniet:initiator01
> cd iqn.2020-01.com.blachniet:initiator01
> set auth userid=username
> set auth password=password
> exit

# Ensure something is listening on 3260
$ ss -napt | grep 3260
LISTEN   0         256                 0.0.0.0:3260             0.0.0.0:*

$ systemctl enable target

# This may not be required
$ firewall-cmd --add-service=iscsi-target --permanent
$ firewall-cmd --reload
```

## Set up the iSCSI initator

This VM has:
- iSCSI initiator
- Docker
- Minikube
- kubectl

```sh
cd initiator
vagrant up
vagrant ssh
```

In the VM:

```sh
kubectl create -f chap-secret.yaml
# Update IP portal and iqn in `iscsi-chap.yaml` if necessary
kubectl create -f iscsi-chap.yaml
```

## Resources

- https://github.com/kubernetes/examples/tree/master/volumes/iscsi
- https://www.server-world.info/en/note?os=Fedora_30&p=iscsi&f=1
- https://www.server-world.info/en/note?os=Fedora_21&p=iscsi&f=2
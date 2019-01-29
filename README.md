# Vagrant Kubernetes MultiNode

## Getting Started

### Prerequisites

You will need to install

* vagrant
* virtualbox

And then, you will need to install the following vagrant plugins:

* vagrant-env
* vagrant-hosts

### Deployment

If you want to start up the kubernetes cluster you only need to exec

```bash
$ vagrant up
```

You can find the kubectl inside the master virtual machine

```bash
$ vagrant ssh master
[vagrant@master ~]$ sudo -s
[root@master vagrant]# kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
master.vagrant.vm    Ready    master   53m   v1.13.1
worker0.vagrant.vm   Ready    <none>   52m   v1.13.1
worker1.vagrant.vm   Ready    <none>   52m   v1.13.1
worker2.vagrant.vm   Ready    <none>   52m   v1.13.1
```

When you want to delete all the virtual machines

```bash
$ vagrant destroy -f
```

## Features

* Kubernetes (1 master, 3+ nodes)

* GlusterFS + Heketi

## License

This project is licensed under the Apache 2 License - see the [LICENSE](LICENSE) file for details


# MPI using Vagrant

This tutorial will walk students through the process of setting up a reasonable environment for MPI programming using [Vagrant](http://www.vagrantup.com).  

## Background

We will deploy a portable cluster with the following configuration:
* Four nodes running Linux CentOS 6.5
* mpich 3.4.1 MPI software

The compute nodes will talk to each other using a private host-only network setup using the following network configuration.
```
node1 192.168.33.1
node2 192.168.33.2
node3 192.168.33.3
node4 192.168.33.4
```

## Install Pre-Requisites

* Download from vagrantup.com the latest version of vagrant software for your operating system.
* Download from virtualbox.org the latest version of virtualbox software for your operating system.

## Setup your compute nodes

* Create a project directory: `mini-cluster`

### Preparing share

```
mkdir share
```

### Preparing node1

* Create a node directory: `mkdir node1`
* Initialize a vagrant node:
```
cd node1
vagrant init puphpet/centos65-x64
```
This process will take awhile the first time around.

* Update the `Vagrantfile` by adding the following lines
```
  config.vm.hostname = "node1"
  config.vm.network "private_network", ip: "192.168.33.1"
  config.vm.synced_folder "../share", "/share"
```
after the line `config.vm.box = "puphpet/centos65-x64"`
* Spin up your **node1** instance: `vagrant up`
 * You might have to run provision if you get the message
```
Machine already provisioned. Run `vagrant provision` or use the `--provision`
```
* Login into your virtual os:
   * If your host os is OSX or Linux you can simply type: `vagrant ssh`
   * If your host os is Windows you'll need to run `putty.exe`
* Generate your ssh key (you can default `enter` on all options):
```
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub /share/node1.pub
```
* Install mpich 3.4.1: `sudo yum -y install mpich mpich-devel`
* Exit the virtual environment: `exit`

### Preparing node2, node3, and node4

* Create directory and initialize another vagrant node:
```
cd ..
mkdir node2
vagrant init puphpet/centos65-x64
```
* Update your `Vagrantfile` by adding the following lines
```
  config.vm.hostname = "node2"
  config.vm.network "private_network", ip: "192.168.33.2"
  config.vm.synced_folder "../share", "/share"
```
after the line `config.vm.box = "puphpet/centos65-x64"`
* Spin up your **node2**: `vagrant up`
* Login into your virtual os
* Generate your ssh key (you can default `enter` on all options):
```
ssh-keygen -t rsa
cp ~/.ssh/id_rsa.pub /share/node2.pub
```
* Install mpich 3.4.1: `sudo yum -y install mpich mpich-devel`
* Exit the virtual environment: `exit`

Now repeat the above steps for `node3` and `node4`.

### Add public keys to each nodes

```
cd ..
cd node1
vagrant ssh
cat /share/node*.pub >> ~/.ssh/authorized_keys
exit
```

Repeat for `node2`, `node3`, and `node4`.

### Destroying a node

When we're done with the work.  Let's gracefully clean up our system.  Make sure that you've exited the guest node first.

```
cd node1
vagrant halt
vagrant destroy
cd ..
```

Now delete the `node1` folder.  The above step is **destructive** and there is no way to undo.

## Your First MPI program

For the following example program , do all your execution on `node1` and out of the `/share` directory.  

```
cd node1
vagrant ssh
cd /share
```

Create a program [hello.c](hello.c) in your `/share` directory.

To compile the program run the following command:

```
/usr/lib64/mpich/bin/mpicc -o hello hello.c
```

Create `hostfile` with the following:

```
192.168.33.1
```

To run your program execute the following command:

```
/usr/lib64/mpich/bin/mpirun -n 4 -f hostfile /share/hello
```

The `hello.c` program demonstrates a simple MPI program that does not perform any communications.  Now, let's do a slightly more complicated program.  For this task, your program will pass a `token` around a ring.  The solution is in the [potato.c](potato.c) code.

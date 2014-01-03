Tested with CentOS 6.4, Vagrant 1.2.7, VirtualBox 4.2.8 on Windows 7 (with GitBash 1.8.1.2).

I tested this locally on my laptop.  You will need the VirtualBox GUI and I copied my final <code>centos64-64</code> Vagrant box up to Google drive to share it out.

1. Download and install VirtualBox

2. Download the CentOS 6.4 Minimal LiveCD ISO

3. Make a new VM in VirtualBox:
* Call it "vagrant-centos64"
* Set OS to Linux and version to Red Hat

4. Boot the VM and install CentOS 6.4 on it:
* Select the LiveCD ISO in storage settings with the IDE controller
* Set the root password to "vagrant":
* Set the hostname to "vagrant-centos64"
* Remove the LiveCD ISO from storage settings after CentOS 6.4 is booted

5. Boot the VM into the CentOS, login to desktop as the root user and update:

<pre>
root@vagrant-centos64$ yum update
</pre>

6. Shutdown the vm and modify the NAT settings:

<pre>
you@host$ cd "C:\Program Files\Oracle\Virtualbox"
you@host$ VBoxManage modifyvm "vagrant-centos64" --natpf1 "guestssh,tcp,,2222,,22"
</pre>

7. Now you can ssh into the VM from a host shell:

<pre>
you@host$ ssh -p 2222 root@127.0.0.1
</pre>

8. Enable the network interface to auto start on the Boot and get dynamic ip, provided by vagrant:

In file:
<pre>
vagrant@vagrant-centos62$ vi /etc/sysconfig/network-scripts/ifcfg-eth0
</pre>

Change:
<pre>
ONBOOT=no
</pre>

To parameters and values:
<pre>
ONBOOT=yes
</pre>

9. Install some stuff on the VM:

<pre>
root@vagrant-centos64$ yum install gcc perl
</pre>

10. Install guest additions, easiest way is using Devices -> Install Guest Additions from VirtualBox GUI, mount, run, and reboot:

<pre>
root@vagrant-centos64$ mount /dev/cdrom /mnt
root@vagrant-centos64$ /mnt/VBoxLinuxAdditions.run
root@vagrant-centos64$ init 6
</pre>

11. Add vagrant user, and configure himfor admin group, from a root shell on the vm:

<pre>
root@vagrant-centos64$ useradd vagrant
root@vagrant-centos64$ passwd vagrant
  vagrant
  vagrant
root@vagrant-centos64$ groupadd admin
root@vagrant-centos64$ usermod -G admin vagrant
</pre>

12. Change the sudoers file, from a root shell on the vm do <code>visudo</code> and:

* Add SSH_AUTH_SOCK to the env_keep option
* Comment out the Defaults requiretty line
* Add the line %admin ALL=NOPASSWD: ALL

The vagrant user should now be able to sudo without typing a password, try sudo ls from vagrant user's shell:

<pre>
vagrant@vagrant-centos62$ sudo ls
</pre>

13. Add vagrant's public key so vagrant user can ssh without password. From vagrant user's shell on vm:

<pre>
vagrant@vagrant-centos62$ mkdir .ssh
vagrant@vagrant-centos62$ curl -k https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub > .ssh/authorized_keys
vagrant@vagrant-centos62$ chmod 0700 .ssh
vagrant@vagrant-centos62$ chmod 0600 .ssh/authorized_keys
</pre>

14. Finally package the box and add it to your vagrant boxes. From vagrant user's shell on vm:

<pre>
vagrant@vagrant-centos64$ sudo yum clean all
</pre>

Shutdown the vm, then from a host shell:

<pre>
you@host$ vagrant package --output centos64-64.box --base vagrant-centos64-64
you@host$ vagrant box add centos64-64 centos64-64.box
</pre>

Be sure the machine named after `--base` is a machine listed displayed in your Virtual Box. If you are creating a new vagrant box from an existing vagrant box, the name may be something like `<dirname>_1369964062`

You should now have a centos62-32 base box in your vagrant boxes:

<pre>
you@host$ vagrant box list
centos64-64
</pre>

Now you can quickly create a Vagrant VM in any directory with:

<pre>
you@host$ /tmp/my_vm $ vagrant init centos62-32
you@host$ /tmp/my_vm $ vagrant up
you@host$ /tmp/my_vm $ vagrant ssh
</pre>

To use the VM on a new host machine, copy the <code>centos64-64.box</code> file to the new host and from the new host's shell run the <code>vagrant box add</code> command on it, now you should see the centos64-64 box in <code>vagrant box list</code> on the new host and you can use the box <code>vagrant init centos64-64</code>.

Tested with CentOS 6.2, Vagrant 1.0.2, VirtualBox 4.1.14 on Ubuntu 10.04 (s031).

I did locally on my laptop as you need the VirtualBox GUI and then copied the final <code>centos62-32.box</code> file up to s031.

1. Download and install VirtualBox

2. Download the CentOS 6.2 LiveCD

3. Make a new VM in VirtualBox:
* Call it "vagrant-centos62-32"
* Set OS to Linux and version to Red Hat
* Go into the new VM's settings and disable audio and usb

4. Boot the VM and install CentOS 6.2 on it:

* Set the root password to "vagrant":
* For the non-root desktop user give username "vagrant" and password "vagrant"
* Set the hostname to "vagrant-centos62"

5. Boot the VM into the CentOS, login to desktop as vagrant user, open terminal, su, install and enable openssh:

<pre>
root@vagrant-centos62$ yum install openssh-server
root@vagrant-centos62$ service sshd start
root@vagrant-centos62$ chkconfig sshd on
root@vagrant-centos62$ netstat -tulpn | grep :22
</pre>

6. Setup port forwarding in virtualbox. Shutdown the vm and from a host shell do:

<pre>
you@host$ VBoxManage modifyvm "vagrant-centos62" --natpf1 "guestssh,tcp,,2222,,22"
</pre>

7. Now you can ssh into the vm from a host shell:

<pre>
you@host$ ssh -p 2222 root@127.0.0.1
</pre>

8. Install some stuff on the VM:

<pre>
root@vagrant-centos62$ yum install nano wget gcc bzip2 make kernel-devel-`uname -r`
</pre>

9. Install guest additions, easiest way is using Devices -> Install Guest Additions in the VM's GUI window.

10. Add vagrant user to admin group, from a root shell on the vm:

<pre>
root@vagrant-centos62$ groupadd admin
root@vagrant-centos62$ usermod -G admin vagrant
</pre>

11. Change the sudoers file, from a root shell on the vm do <code>visudo</code> and:

* Add SSH_AUTH_SOCK to the env_keep option
* Comment out the Defaults requiretty line
* Add the line %admin ALL=NOPASSWD: ALL

The vagrant user should now be able to sudo without typing a password, try sudo ls from vagrant user's shell:

<pre>
vagrant@vagrant-centos62$ sudo ls
</pre>

12. Add vagrant's public key so vagrant user can ssh without password. From vagrant user's shell on vm:

<pre>
vagrant@vagrant-centos62$ mkdir .ssh
vagrant@vagrant-centos62$ curl -k https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub > .ssh/authorized_keys
vagrant@vagrant-centos62$ chmod 0700 .ssh
vagrant@vagrant-centos62$ chmod 0600 .ssh/authorized_keys
</pre>

13. Enable the network interface to auto start on the Boot and get dynamic ip, provided by vagrant:

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
NM_CONTROLLED=yes
BOOTPROTO=dhcp
</pre>

14. Finally package the box and add it to your vagrant boxes. From vagrant user's shell on vm:

<pre>
vagrant@vagrant-centos62$ sudo yum clean all
</pre>

Shutdown the vm, then from a host shell:

<pre>
you@host$ vagrant package --output centos62-32.box --base vagrant-centos62-32
you@host$ vagrant box add centos62-32 centos62-32.box
</pre>

Be sure the machine named after `--base` is a machine listed displayed in your Virtual Box. If you are creating a new vagrant box from an existing vagrant box, the name may be something like `<dirname>_1369964062`

You should now have a centos62-32 base box in your vagrant boxes:

<pre>
you@host$ vagrant box list
centos62-32
</pre>

Now you can quickly create a Vagrant VM in any directory with:

<pre>
you@host$ /tmp/my_vm $ vagrant init centos62-32
you@host$ /tmp/my_vm $ vagrant up
you@host$ /tmp/my_vm $ vagrant ssh
</pre>

To use the VM on a new host machine, copy the <code>centos62-32.box</code> file to the new host and from the new host's shell run the <code>vagrant box add</code> command on it, now you should see the centos62-32 box in <code>vagrant box list</code> on the new host and you can use the box <code>vagrant init centos62-32</code>.

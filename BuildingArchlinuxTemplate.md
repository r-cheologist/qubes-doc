---
layout: wiki
title: BuildingArchlinuxTemplate
permalink: /wiki/BuildingArchlinuxTemplate/
---

Template building
=================

The archlinux VM is now almost working as a NetVM. Based on qubes-builder code, you could find below how to build it and problem that could arise from template building to using archlinux as a netvm:

Download qubes-builder git code
-------------------------------

Prefer marmarek git repository as it is the most recent one

Change your builder.conf
------------------------

Change the following variables GIT\_SUBDIR=marmarek DISTS\_VM=archlinux

Get all required sources
------------------------

``` {.wiki}
make get-sources
```

Make all required qubes components (the first use of the builder can take several hours depending on your bandwidth as it will install an archlinux chroot):
------------------------------------------------------------------------------------------------------------------------------------------------------------

``` {.wiki}
make vmm-xen-vm
make core-vchan-xen-vm
make linux-utils-vm
make core-agent-linux-vm
make gui-common-vm
make gui-agent-linux-vm
```

Now build the template itself
-----------------------------

This can take again several hours, especially the first time you built and archlinux template:

``` {.wiki}
make linux-template-builder
```

Retrieve your template
----------------------

You can now find your template in qubes-src/linux-template-builder/rpm/noarch. Install it in dom0 (just take care as it will replace your current archlinux-x64 template)

* * * * *

Known problems during building or when running the VM
=====================================================

xen-vmm-vm fail to build with a PARSETUPLE related error:
---------------------------------------------------------

Commenting out "\#define HAVE\_ATTRIBUTE\_FORMAT\_PARSETUPLE" from chroot\_archlinux/usr/include/python2.7/pyconfig.h fixes the problem, but it isn't the right solution [1]...

A better fix is planned for the next python release (the bug is considered release blocking), and will be updated in archlinux chroot as soon as available.

[1] [​http://bugs.python.org/issue17547](http://bugs.python.org/issue17547)

Qubes-OS is now using different xenstore variable names, which makes to archlinux VM failing to boot
----------------------------------------------------------------------------------------------------

Apply the following fix in the template to revert the variable name to the old Qubes version (eg: sudo mkdir /mnt/vm; sudo mount /var/lib/qubes/vm-templates/archlinux-x64/root.img /mnt/vm; sudo chroot /mnt/vm then apply the fix, then umount /mnt/vm)

``` {.wiki}
sudo sed 's:qubes-keyboard:qubes_keyboard:g' -i /etc/X11/xinit/xinitrc.d/qubes-keymap.sh

sudo sed 's:qubes-netvm-domid:qubes_netvm_domid:g' -i /etc/NetworkManager/dispatcher.d/30-qubes-external-ip
sudo sed 's:qubes-netvm-external-ip:qubes_netvm_external_ip:g' -i /etc/NetworkManager/dispatcher.d/30-qubes-external-ip

sudo sed 's:qubes-netvm-network:qubes_netvm_network:g' -i /usr/lib/qubes/init/network-proxy-setup.sh
sudo sed 's:qubes-netvm-gateway:qubes_netvm_gateway:g' -i /usr/lib/qubes/init/network-proxy-setup.sh
sudo sed 's:qubes-netvm-netmask:qubes_netvm_netmask:g' -i /usr/lib/qubes/init/network-proxy-setup.sh
sudo sed 's:qubes-netvm-secondary-dns:qubes_netvm_secondary_dns:g' -i /usr/lib/qubes/init/network-proxy-setup.sh

sudo sed 's:qubes-vm-type:qubes_vm_type:g' -i /usr/lib/qubes/init/qubes-sysinit.sh

sudo sed 's:qubes-ip:qubes_ip:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-netmask:qubes_netmask:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-gateway:qubes_gateway:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-secondary-dns:qubes_secondary_dns:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-netvm-network:qubes_netvm_network:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-netvm-gateway:qubes_netvm_gateway:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-netvm-netmask:qubes_netvm_netmask:g' -i /usr/lib/qubes/setup-ip
sudo sed 's:qubes-netvm-secondary-dns:qubes_netvm_secondary_dns:g' -i /usr/lib/qubes/setup-ip

sudo sed 's:qubes-iptables-domainrules:qubes_iptables_domainrules:g' -i /usr/sbin/qubes-firewall
sudo sed 's:qubes-iptables-header:qubes_iptables_header:g' -i /usr/sbin/qubes-firewall
sudo sed 's:qubes-iptables-error:qubes_iptables_error:g' -i /usr/sbin/qubes-firewall
sudo sed 's:qubes-iptables:qubes_iptables:g' -i /usr/sbin/qubes-firewall

sudo sed 's:qubes-netvm-domid:qubes_netvm_domid:g' -i /usr/sbin/qubes-netwatcher
sudo sed 's:qubes-netvm-external-ip:qubes_netvm_external_ip:g' -i /usr/sbin/qubes-netwatcher
```

The nm-applet (network manager icon) fails to start when archlinux is defined as a template-vm:
-----------------------------------------------------------------------------------------------

In fact /etc/dbus-1/system.d/org.freedesktop.[NetworkManager?](/wiki/NetworkManager).conf does not allow a standard user to run network manager clients. To allow this, one need to change inside \<policy context="default"\>:

``` {.wiki}
<deny send_destination="org.freedesktop.NetworkManager"/>
```

to

``` {.wiki}
<allow send_destination="org.freedesktop.NetworkManager"/>
```

DispVM, Yum proxy and most Qubes addons (thunderbird ...) have not been tested at all.
--------------------------------------------------------------------------------------
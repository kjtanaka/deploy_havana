.. Simple Deploy OpenStack Havana documentation master file, created by
   sphinx-quickstart on Wed Oct 16 15:15:10 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Add Nova Compute Nodes
==========================================================

::

    Public Network
   +--------------------------------------------------------------------------
       |                                 |                                |
       | eth1 [xxx.xxx.xxx.xxx/24]       | eth1 [xxx.xxx.xxx.xxx/24]      |
     +-----------------+               +-----------------+              +-----
     | host01          |               | host02          |              |
     | =============== |               | =============== |              |
     | Keystone        |               | Nova Compute    |              |
     | Glance          |               | Nova Network    |              |
     | Horizon         |               |                 |              |
     | Nova API        |               |                 |              |
     | Nova Scheduler  |               |                 |              |
     | Nova Compute <----[disable]     |                 |              |
     | Nova Network    |               |                 |              |
     | Nova Conductor  |               |                 |              |
     +-----------------                 -----------------               +-----
       | eth0 [xxx.xxx.xxx.xxx/24]       | eth0 [xxx.xxx.xxx.xxx/24]      |
       |                                 |                                |
   +--------------------------------------------------------------------------
    Internal/Admin Network

First, update ``/etc/hosts`` if needed, and copy it and your ``deploy_havana``
directory to the compute node(host02 in this example). ::

   scp /etc/hosts host02:/etc/hosts
   scp -r deploy_havana host02:deploy_havana

Login to host02 and execute ``add_nova_compute.sh`` with ``-ex`` option ::

   ssh host02
   cd deploy_havana
   bash -ex add_nova_compute.sh

If the bash script finshed fine, host02 would be rebooted.
While waiting for host02 to be online, delete all instances and disable Nova Compute. ::

   nova list
   nova delete <instance 1> <instance 2> ...
   nova-manage service disable --host host01 --service nova-compute

It is not necessary to disable Nova Compute, but Nova Compute creates high-load, 
so disablling it is wise. You should regard host01 as the management host from here.

Check service list. ::

   nova-manage service list

   Binary           Host                                 Zone             Status     State Updated_At
   nova-scheduler   host01                               internal         enabled    :-)   2013-10-17 03:31:11
   nova-conductor   host01                               internal         enabled    :-)   2013-10-17 03:31:07
   nova-consoleauth host01                               internal         enabled    :-)   2013-10-17 03:31:10
   nova-cert        host01                               internal         enabled    :-)   2013-10-17 03:31:10
   nova-network     host01                               internal         enabled    :-)   2013-10-17 03:31:05
   nova-compute     host01                               nova             disabled   :-)   2013-10-17 03:31:07
   nova-console     host01                               internal         enabled    :-)   2013-10-17 03:31:03
   nova-compute     host02                               nova             enabled    :-)   2013-10-17 03:31:03
   nova-network     host02                               internal         enabled    :-)   2013-10-17 03:31:09

If the states of host02's services become ``:-)``, boot an instance. ::

   nova boot --image ubuntu-12.04 --flavor m1.small --key-name key1 vm001
   nova list
   nova console-log vm001
   ssh -l ubuntu -i key1.pem 192.168.201.3

If you have more hosts, add some more compute nodes with the same procedure, 
and create your first cluster on the Cloud.

.. Simple Deploy OpenStack Havana documentation master file, created by
   sphinx-quickstart on Wed Oct 16 15:15:10 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Build All-in-One Node Anyway
=============================

::

    Public Network
   +-------------------------------
       |                           
       | eth1 [xxx.xxx.xxx.xxx/24] 
     +-----------------+           
     | host01          |           
     | =============== |           
     | Keystone        |           
     | Glance          |           
     | Horizon         |          
     | Nova API        |          
     | Nova Scheduler  |          
     | Nova Compute    |
     | Nova Network    | 
     | Nova Conductor  |      
     +-----------------           
       | eth0 [xxx.xxx.xxx.xxx/24] 
       |                           
   +-------------------------------
    Internal/Admin Network

Quick Installation by Bash Script
---------------------------------

First of all, you need to update ``/etc/hosts`` for your environment. If your hostname is not resolvable,
``nova-network`` fails to start.

Switch to ``root`` user, and download the bash script. ::

   sudo -i
   git clone https://github.com/kjtanaka/deploy_havana.git
   cd deploy_havana

Create and modify your ``setuprc`` file. Two examples of ``setuprc`` are given below. ::

   cp setuprc_example_[1NIC/2NICs] setuprc
   vi setuprc

* `setuprc example for 2NICs <https://github.com/kjtanaka/deploy_havana/blob/master/doc/setuprc_2nics.rst>`_
* `setuprc example for 1NIC <https://github.com/kjtanaka/deploy_havana/blob/master/doc/setuprc_1nic.rst>`_

Execute ``build_all_in_one.sh`` with ``-ex`` option. ::

   bash -ex build_all_in_one.sh

If the bash script finished fine, your machine would be rebooted at the end of the script. 
So wait until it becomes online.

Boot Your First Instance and Feel It
------------------------------------

Load nova environment. ::

   cd deploy_havana
   source admin_credential

Check the list of images, flavors and keypairs. ::

   nova image-list

   +--------------------------------------+--------------+--------+--------+
   | ID                                   | Name         | Status | Server |
   +--------------------------------------+--------------+--------+--------+
   | d95da7a5-d67d-4569-9edf-9524071b2c05 | ubuntu-12.04 | ACTIVE |        |
   +--------------------------------------+--------------+--------+--------+
   
::

   nova flavor-list

   +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
   | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public |
   +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+
   | 1  | m1.tiny   | 512       | 1    | 0         |      | 1     | 1.0         | True      |
   | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      |
   | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      |
   | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      |
   | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      |
   +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+

::

   nova keypair-list

   +------+-------------------------------------------------+
   | Name | Fingerprint                                     |
   +------+-------------------------------------------------+
   | key1 | xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx |
   +------+-------------------------------------------------+

Boot your first instance. ::

   nova boot --image ubuntu-12.04 --flavor m1.small --key-name key1 vm001

Check the status of instance. ::

   nova list

   +--------------------------------------+-------+--------+------------+-------------+-----------------------+
   | ID                                   | Name  | Status | Task State | Power State | Networks              |
   +--------------------------------------+-------+--------+------------+-------------+-----------------------+
   | a94c8354-b2fb-432e-8ff2-d98e30b50162 | vm001 | ACTIVE | None       | Running     | private=192.168.201.2 |
   +--------------------------------------+-------+--------+------------+-------------+-----------------------+

Take a look at the console log. ::

   nova console-log vm001

If all look good, you should be able to ssh to your first instance. ::

   ssh -i key1.pem ubuntu@192.168.201.2

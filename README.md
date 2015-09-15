# xslab - XenServer Lab Script
xslab is a script to ease the use of snapshots for testing / lab environments which require regular reverting of virtual machines.

Motivation
===========
The main goal of this script is to create some sort of tool which automatically resets / reverts a specific set of powered off machines. To achieve this, virtual machines can be added to the lab and several baselines can be created.

A cronjob can be installed to periodically revert stopped VMs. This way users can initiate the rollback of lab machines, without requiring shell or management access to the XenServer pool.

Installation
===========
To use xslab, xslab itself and a cronjob have to be installed.

xslab installation
-------------------

Download xslab and copy it to $PATH on your XenServer instance.

You can use the following steps:

    mkdir -p /root/bin/
    wget https://raw.githubusercontent.com/fetzerms/xslab/master/xslab -O /root/bin/xslab
    chmod +x /root/bin/xslab

Caution: Always check the actual file contents before executing the script. 
This applies to any downloaded bash scripts which will be executed as the root users ;)

cronjob installation
---------------------

To enable the auto-revert feature, a cronjob has to be installed. If you do not want to use the auto-revert feature, the cronjob installation can be left out.

Use the following steps to install the cronjob:

    crontab -e
Add the following cronjob (you may use other intervals): 
    */5	*	*	*	*	/root/bin/xslab auto-revert
 
General Usage
==============

The usage is as follows:

    xslab <command> <parameters>

Commands:
- add               : Add a vm to xslab. This will create an initial baseline.
- delete            : Delete a vm from xslab. This will NOT delete baselines.
- enable            : Enable a VM which has been added to xslab.
- disable           : Disable a VM which has been added to xslab.
- list              : List VMs that have been added to xslab.
- revert            : Revert VM to latest baseline.
- auto-revert       : Revert all enabled and powered off VMs to the latest baseline and boot them.
- baseline-create   : Create a new baseline for a given VM.
- baseline-delete   : Delete the n-th baseline from a given VM.
- baseline-list     : Lists all baselines for a given VM.

Parameters:
- --vm/-m       : The VM name to perform operations on.
- --num/-n      : The n-th element to perform operations on. 
                       Currently only used with baseline-delete

Examples:
- xslab list
- xslab add --vm lab_debian8
- xslab baseline-create --vm lab_debian8
- xslab baseline-delete --vm lab_debian8 --num 1
- xslab baseline-list --vm lab_debian8
- xslab revert --vm lab_debian8
- xslab auto-revert
- xslab delete --vm lab_debian8

Example usage
--------------
Lets assume the following scenario:
- We have a set of virtual machines which we cant to add to xslab.
- The VMs are called lab_debian_8, lab_debian_7, lab_debian_6
- So we add each vm to the lab: 
  - xslab add --vm lab_debian_8
  - xslab add --vm lab_debian_7
  - xslab add --vm lab_debian_6
- As debian 6 is quite old, we decide that we do not want to do any reverts on it. Hence we disable lab_debian_6
  - xslab disable --vm debian_6
- Any change to debian_8 and debian_7 will be reverted if we power the machine down.
- At a certain point we want to "pin" the state of the vms, e.g. if we want to make sure that a specific software is included in all restored vms. Hence we install the required software and create a new baseline for the vms afterwards:
  - xslab baseline-create --vm lab_debian_8
  - xslab baseline-create --vm lab_debian_7
- Now the vms will reset to this baseline each time they are powered off.

Contributing
============
Contributions and feature requests are always welcome!

If you have any additions, examples or bugfixes ready, feel free to create a pull request on GitHub. The pull requests will be reviewed and will be merged as soon as possible. To ease the process of merging the pull requests, please create one pull request per feature/fix, so those can be selectively included in xslab.

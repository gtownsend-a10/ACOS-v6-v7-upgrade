##################
Upgrading to ACOS 7.x.x
##################

*********
Overview
*********

The Thunder device is provided with preinstalled ACOS software along with an ADC license. When you power ON the device, it boots up with the preinstalled software. To access the latest new features and software fixes as they become available, you must upgrade the ACOS software. 

If you are a new ACOS user, check the following documentation on the A10 Documentation Site:  
  * For instructions on installing new hardware, see Installation Guide for Thunder Physical Appliance. 
  * For instruction on installing vThunder, see Installation Guide for Thunder Virtual Appliance. 
  * For instructions on installing cThunder, see Installation Guide for Thunder Container. 
  * For instructions on installing ACOS on Bare Metal, see Installation Guide for Bare Metal. 
  * For instructions on acquiring a product license, see Global Licensing Manager.  
  * For initial configuration instructions and quick processes handbook, see Quick Start Guide. 

Purpose 
=======

This guide provides detailed instructions for upgrading from ACOS 4.x or 5.x to the latest version of 6.x. It includes information on pre-upgrade preparations, the upgrade procedure, post-upgrade tasks, troubleshooting tips, and additional resources. 

The following topics are covered:  

* General Guidelines
* Prerequisites
* Pre-Upgrade Tasks 
* Upgrade Instructions 
* Post-Upgrade Tasks 
* Rollback Upgrade 

General Guidelines 
******************

Consider the following recommendations before upgrading the ACOS device: 

* Test the upgrade procedure in a non-production environment to ensure its effectiveness. 

* ACOS device is upgraded by copying the software image to your device or other system on your local network and then upgrading the device using the CLI or GUI instructions. 

* Regardless of whether you have an ADC, CGN, or TPS, a single software image is used to upgrade your ACOS device. However, ensure that the correct product license is obtained and activated.  

.. note::
   For TPS upgrade instructions, see TPS Upgrade Guide.
 
* During the reboot, the system performs a full reset and will be offline. The actual duration may vary depending on the system parameters.  

Unsupported Hardware and Features
*********************************

.. warning:: 
   * The 3rd Generation Hardware Platforms cannot be upgraded to ACOS 6.x version. For more information, see Hardware Platforms Support.  
   * The Web Application Firewall (WAF) is no longer supported starting from the ACOS 6.x release. Hence, all WAF configurations will be removed after the upgrade. For more information, see Web Application Firewall Changes. 

Prerequisites 
*************

This section outlines essential information that you should know before proceeding with the upgrade process.  

Table 1 : Prerequisite Tasks 

+----------------------------------------------------------------+-----------------------------+
| Tasks                                                          | Refer                       |
+----------------------------------------------------------------+-----------------------------+
| Check the platform compatibility versus the supported          | Hardware Platforms Support  |
| release version                                                |                             |
+----------------------------------------------------------------+-----------------------------+
| Check the SKUs or product licenses availability.               | Hardware Product Licenses   |
+----------------------------------------------------------------+-----------------------------+
| Check the storage and memory requirement.                      | System Requirement          |
+----------------------------------------------------------------+-----------------------------+
| Carefully review the new features, known issues, and changes   | Documentation Site          |
| to default behavior.                                           |                             |
+----------------------------------------------------------------+-----------------------------+
| Understand what the ACOS partitions and how to take a backup.  | System Partitions           |
+----------------------------------------------------------------+-----------------------------+
| Check the instructions for taking a system backup.             | Perform a Backup            |
+----------------------------------------------------------------+-----------------------------+
| Download the ACOS software image.                              | Download Software Image     |
+----------------------------------------------------------------+-----------------------------+


.. note::
   * Schedule a maintenance window for the upgrade, considering the potential downtime required. Communicate this schedule to relevant stakeholders. 
   * Inform all users about the scheduled downtime and ensure they save any unsaved work or log out of the system before the upgrade begins. 

Upgrade Requirements
********************

Version Requirements
====================

This upgrade document is for a 5.2.x to 6.x, Refer to the upgrade documentation for the specific upgrade path.

The following section helps you in identifying the upgrade paths to the latest versions of ACOS releases:

The upgrade path Table lists the supported upgrade paths for ACOS releases:


+----------------+----------------+----------------+----------------+
|Existing Version|First Hop       |Second Hop      |Third Hop       |
+----------------+----------------+----------------+----------------+
|4.1.x           |4.1.x to 5.x    |5.x to 6.0.7    | 6.0.7 to 7.0.x |
+----------------+----------------+----------------+----------------+
|5.1.x           |5.1.x to 5.2.x  |5.2.x to 6.0.7  | 6.0.7 to 7.0.x |
+----------------+----------------+----------------+----------------+
|5.2.x           |5.2.x to 6.0.7  | 6.0.7 to 7.0.x | 6.0.7 to 7.0.x |
+----------------+----------------+----------------+----------------+
|6.0.x           |6.0.x to 6.0.7  | 6.0.7 to 7.0.x |                |
+----------------+----------------+----------------+----------------+

System Requirement 
==================
The system requirements for ACOS software include the following: 

ACOS 6.x
  * The minimum disk space requirement is 30 GB.  
  * The minimum memory requirement is 8 GB. 

ACOS 7.x
  * The minimum disk space requirement is 128 GB.  
  * The minimum memory requirement is 16 GB. 



System Partitions
=================

Each ACOS device contains one shared partition. By default, this is the only partition on the device and cannot be deleted. If there are no additional partitions on the device, all configuration changes occur in the shared partition. 

You can save the configuration of these partitions to either the default startup-config, the current, or a new configuration profile. To save the partition configuration, use the write memory command. 

Depending on the configuration profile and the partition being saved to, the following summarizes the write memory command usage: 

 
+------------------------------------------------+--------------------------------------------------------------+
| Command                                        | Descriptions                                                 |
+------------------------------------------------+--------------------------------------------------------------+
| `write memory`                                 | Save the running configuration to the startup-config or the  |
|                                                | current profile in the current partition.                    |
+------------------------------------------------+--------------------------------------------------------------+
| `write memoroy all-partitions`                 | Save the running configuration to their respective           |
|                                                | startup-config or their current profiles of all partitions.  |
|                                                | partition.                                                   |
+------------------------------------------------+--------------------------------------------------------------+
| `write memory <profile name>`                  | Save the running configuration to the new profile            |
|                                                | in the current partition                                     |
+------------------------------------------------+--------------------------------------------------------------+
| `write memory <profile name> all-partitions`   | Save the running configuration to the new profile            |
|                                                | of all partitions                                            |
+------------------------------------------------+--------------------------------------------------------------+

Review Boot Order 
=================

This section describes general guidelines on how ACOS selects the boot image. 

Each ACOS device contains multiple locations where software images can be placed. The "Upgrade Process" table provides an overview of the general upgrade process. 

  * When you load a new image onto the ACOS device, you can select the image device (disk or CF) and the area (primary or secondary) on the device.  

  * When you power ON or reboot the ACOS device, it always attempts to boot from the disk, using the image area specified in the configuration (primary disk, by default). If a disk fails, the device attempts to boot from the same image area on the backup disk (if applicable to the device model). 

You need to change the boot order only when you plan to upload the new image into an image area other than the first image area the ACOS device uses when it boots (primary disk). To change the boot order, use the bootimage command.  

.. note::

  A10 Networks recommends installing the new image into the inactive disk image area, either primary or secondary, while retaining the old image in the other area. This helps to 
  restore the system in case a downgrade is necessary or if an issue occurs while rebooting the new image.  

Upgrade Process
===============

+-------------+--------------+---------+------------+
|System       | Partition 1  | Upgrade | Partition 2|
+-------------+--------------+---------+------------+
|New System   | Acitve       |         | Inactive   |
+-------------+--------------+---------+------------+
|1st Upgrade  | Active       |   -->   | Inactive   |
+-------------+--------------+---------+------------+
|2nd Upgrade  | Inactive     |   <--   | Active     |
+-------------+--------------+---------+------------+
|Next Upgrade | Active       |   -->   | Inactive   |
+-------------+--------------+---------+------------+
|Next Upgrade | Inactive     |   -->   | Active     |
+-------------+--------------+---------+------------+

Download Software Image 
=======================

A10 Networks has two device types, FTA and non-FTA.  All vThunder devices will use the non-FTA version and depending on the hardware type will determin the correct image.  To determine if your device has an FTA, login to the device and run the following command:

  .. code-block:: shell

     ACOS# show hardware | inc FPGA

If a response is shown then the device had and FTA.

.. code-block:: shell

   FPGA       : 4 instance(s) present

if the device does not have an FTA, no response to the ``show hardware`` command is displayed

Log in to A10 Networks Support using the GLM credential and download the ACOS upgrade package as specified below:  

* For FTA enabled platforms, use the image with the file name:

  .. code-block:: shell
 
    ACOS_FTA_<version>.upg

* For Non-FTA enabled platforms (including vThunder), use the image with the file name: 

  .. code-block:: shell

    ACOS_non_FTA_<version>.upg

Perform a Backup 
================

It's essential to perform a complete backup of your data, including configuration settings, databases, and any customizations. This backup will prove invaluable in case of unexpected issues during the upgrade and you want to restore it. For information about restoring a backup, see Restore from a Backup.  

This section provides examples of how to back up your system. 

CLI Configuration Backup 
========================

It is recommended to backup the system and the log files prior to upgrading the software.  
* The following example creates a backup of the system (startup-config file, aFleX scripts, and SSL certificates and keys) on a remote server using SCP:

  .. code-block:: shell

    ACOS(config)# backup system scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backupfile.tar.gz

* The following example creates a daily backup of the log entries in the syslog buffer. The connection to the remote server will be established using SCP on the management interface (use-mgmt-port).  

   .. code-block:: shell

     ACOS(config)# backup log period 1 use-mgmt-port scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backuplog.tar.gz

GUI Configuration Backup
========================

1. Log in to ACOS Web GUI using your credentials. 

2. Navigate to System >> Maintenance >> Backup.  
   >  == Add screenshot? 

Pre-Upgrade Tasks 
*****************

Before upgrading ACOS software, you must perform some basic checks. Keep the below information handy to ensure a seamless upgrade.  
For an automated script to check the system requirements, [ACOS v7 Upgrade Check](https://github.com/gtownsend-a10/ACOS_v7_upgrade_check)

Upgrade Preparation Checklist 
=============================

  * Verify platform compatability:

  .. code-block:: shell

    ACOS(config)# show hardware | inc Gateway
  
  Validate the platform is supported on version 7.x

  * vThunder:

  .. code-block:: shell

     Thunder Series Unified Application Service Gateway vThunder
        
  * Hardware:

   .. code-block:: shell

     Thunder Series Unified Application Service Gateway TH5840S
      
  * Check the current software version

    .. code-block:: shell

      ACOS\>show version | inc ACOS
  
    Validate that the current version is 6.0.7 or later.

    .. code-block:: shell

      64-bit Advanced Core OS (ACOS) version 5.2.1-p5, build 114 (Jul-14-2022,05:11)
  
  * Check the current system disk space and verify minimum disk requriements 

    .. code-block:: shell

       ACOS(config)#show disk
       Total(MB)    Used(MB)       Free(MB)       Usage
       ---------------------------------------------------
       20480          10421          10058          50%
       Hard Disk Primary Status : OK
          
  * Check Memory: 

    .. code-block:: shell

      ACOS(config)#show memory | inc Memory
      Memory:  8127392      4742619     3384773   58.30%

      Verify minimum memory requriements, from first column:
  * Check the system boot order to determine new destination:
  
    .. code-block:: shell

      ACOS(config)#show bootimage | inc *
      Hard Disk primary         5.2.1-p5.114 (*)

    This will display the current Default boot location

   * Save all primary, secondary, and partition configurations

   .. code-block:: shell

      write memory all-partitions 
      Building configuration...
      Write configuration to default primary startup-config
      Write configuration to profile "pri_default" on partition GSLB 
      [OK]
  
  * Backup the system configuration

   .. code-block:: shell

      ACOS(config)# backup system scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backupfile.tar.gz

  * Backup system log files

   .. code-block:: shell

      ACOS(config)# backup log period 1 use-mgmt-port scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backuplog.tar.gz
 
   .. note::   

      For detailed information on all the commands, see ***Command Line Interface Reference***.

Download Software Image 
=======================

A10 Networks has two device types, FTA and non-FTA.  All vThunder devices will use the non-FTA version and depending on the hardware type will determin the correct image.  The preupgrade scirpt will report FTA or non-FTA.  You can determine if your device has an FTA, login to the device and run the following command:

  .. code-block:: shell

     ACOS# show hardware | inc FPGA

If a response is shown then the device had and FTA.

.. code-block:: shell

   FPGA       : 4 instance(s) present

if the device does not have an FTA, no response to the ``show hardware`` command is displayed

Log in to A10 Networks Support using the GLM credential and download the ACOS upgrade package as specified below:  

* For FTA enabled platforms, use the image with the file name:

  .. code-block:: shell
 
    ACOS_FTA_<version>.upg

* For Non-FTA enabled platforms (including vThunder), use the image with the file name: 

  .. code-block:: shell

    ACOS_non_FTA_<version>.upg

Upgrade Instructions
********************

This section describes the upgrade instructions using CLI and GUI. The upgrade instruction provided in this section applies to FTA platforms, non-FTA platforms, and non-aVCS environments.  

CLI Configuration 
=================

1. Creates a backup of the system (startup-config file, aFleX scripts, and SSL certificates and keys) on a remote server using SCP:

  .. code-block:: shell

    ACOS(config)# backup system scp://exampleuser@192.168.3.3/home/users/exampleuser/backups/backupfile.tar.gz

2. Upgrade the ACOS device to the inactve partition.  

  * If the primary hard disk is active upgrade the secondary hard disk: 

   .. code-block:: shell

      ACOS-5-x(config)# upgrade hd sec scp://2.2.2.2/images/ACOS_<version>.upg
     
   .. note::

      Use the approprate FTA or non-FTA ACOS version identified in the Upgrade Preparation Checklist

  * If the secondary hard disk is active upgrade the primary hard disk:

   .. code-block:: shell

     ACOS-5-x(config)# upgrade hd pri scp://2.2.2.2/images/ACOS_<version>.upg

   .. note::

    Use the approprate FTA or non-FTA ACOS version identified in the Upgrade Preparation Checklist   

3. You will be prompted to reboot your ACOS device
    
   ==Choose "NO"==
  
4. You will be prompted to reboot your ACOS device. 

5. Press yes to reboot and bring up the upgraded ACOS software.  
  
   .. note:: 
     Allow up to five minutes for the reboot to complete. (The typical reboot time is 2-3 minutes.) 

6. Import the required license and reboot again.  
  The upgrade process is completed successfully.  

Post-Upgrade Tasks 
******************

After performing upgrade, it is important to perform some basic post-upgrade checks.  

Table 3 : Post-Upgrade Checklist 

+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
|Tasks                                                               |Command or Action                                                            |
+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
|Verify that the upgrade was successfully                            |ACOS>`show version`                                                          | 
+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
|Verify the required license is imported successfully                |ACOS(config)#`show license-info`                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
|Verify if the saved configuration from all the                      |ACOS(config)#`startup-config [all \| all-partitions \| partition \| profile]`|
|partitions are loaded successfully                                  |                                                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
|Conduct thorough functional testing to ensure that all core         |NA                                                                           |
|features and functionalities work as expected in the latest version |                                                                             |
+--------------------------------------------------------------------+-----------------------------------------------------------------------------+
 

Rollback Upgrade 
*********

In case the upgrade encounters significant issues or if it fails, have a rollback plan ready to revert to the previous version. The rollback for ACOS device is similar to the upgrade process.  

**Table 4 : Rollback Tasks**

Tasks
Refer 
Carefully review the restoring the system backup information.
Key Considerations for System Restore 

Download your current version ACOS software image.
Download Software
Perform the upgrade instructions
Upgrade Instructions  
Restore the backed up configurations
Restore Example 
Perform the post-upgrade tasks
Post-Upgrade Tasks 

 

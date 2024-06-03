

# Prospoed integration of Rasdaemon and PCIeD for SONiC 4.4.0

## High Level Design Document
**Rev 0.1**

## Table of Contents

* [Revision](#revision)
* [About This Manual](#about-this-manual)
* [Introduction](#Introduction)
* [Rasdaemon Integration in SONiC 4.4.0](#Rasdaemon-Integration)
* [PCIeD Integration in SONiC 4.4.0](#PCIeD-Integration)


# Revision

Rev   |   Date   |  Author   | Change Description
:---: | :-----:  | :------:  | :---------
0.1   | 05/16/24 | Precy Lee | Initial version
        


# About this Manual

This document describes the proposed integration of Rasdaemon and PCIeD in SONiC 4.4.0. 

# Introduction

Rasdaemon is a RAS (Reliability, Avaliability and Serviceability) logging tool. It helps in monitoring hardware errors such as memory errors, PCIe errors, and more, using the EDAC tracing events. PCIeD is a tool designed for monitoring and managing PCI Express devices. It provides detailed information and error reporting for PCIe devices, aiding in the detection and resolution of hardware issues. Integrating rasdaemon and PCIeD with SONiC 4.4.0 enhances the system's ability to detect, log, and respond to hardware errors. This leads to improved system reliability, faster troubleshooting, and better overall performance.

# Rasdaemon Integration in SONiC 4.4.0

## Current Behaviors 

	root         648       1  0 21:46 ?        00:00:00 /usr/sbin/rasdaemon -f -r
 
* Rasdaemon logs messages to syslog
* Kernel error messages are logged to dmesg. 
* Event errors are recorded in the database. The Sqlite3 database is located at /var/lib/rasdaemon/ras-mc_event.db 
* The database can be examined using 'ras-mc-ctl --errors'

### System Boot  

The Sqlite3 database serves as a persistent record of the RAS events after system reboots. 

### Image Upgrade

The Sqlite3 database does not persist through image upgrades.  


## Work Proposed for SONiC 4.4.0

Technical Support (TS) currently collects rasdaemon syslog and dmesg infomration. To enhance this, TS should also gather information from the SQLite3 database using 'ras-mc-ctl' outputs.

### Tasks to be Completed

* Enhance TS to Collect 'ras-mc-ctl' Outputs:

		root@sonic:/home/admin# ras-mc-ctl --errors
		root@sonic:/home/admin# ras-mc-ctl --error-count
		root@sonic:/home/admin# ras-mc-ctl --summary

* Fix the ras-mc-ctl --errors Defect
	
		root@sonic:/home/admin# ras-mc-ctl --errors
		No Memory errors.
		No PCIe AER errors.
		No Extlog errors.
		DBD::SQLite::db prepare failed: no such table: devlink_event at /usr/sbin/ras-mc-ctl line 1306.
		Can't call method "execute" on an undefined value at /usr/sbin/ras-mc-ctl line 1307.


## Machine Check Errors Injection Test

'mce-inject' allows to inject machine check errors on the software level into a running Linux kernel. This is intended for validation of the kernel machine check handler.

* UT: Log two corrected machine checks

		# mce-inject test/corrected

		# dmesg
		 [1395697.527759] Starting machine check poll CPU 0
		 [1395697.527806] Machine check poll done on CPU 0
		 [1395697.527958] mce: [Hardware Error]: Machine check events logged
		 [1395697.528035] Starting machine check poll CPU 1
		 [1395697.528075] Machine check poll done on CPU 1
		 [1395697.528170] mce: [Hardware Error]: Machine check events logged

		#syslog
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]:            <...>-30397 [4275616]     0.137781: mce_record:           		 2024-05-16 10:21:24 -0700 bank=2, status= 9400000000000000, No Error, mci=Corrected_error Error_enabled, mca=No Error, cpu_type= 		Intel generic architectural MCA, cpu= 1, socketid= 1, addr= 1234, mcgstatus=0, mcgcap= f000814, apicid= 20
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]: cpu 00:rasdaemon: mce_record store: 0x1d3ab48
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]: rasdaemon: register inserted at db
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]:            <...>-15467 [4275616]     0.139593: mce_record:           		 2024-05-16 15:23:24 -0700 bank=1, status= 9400000000000000, No Error, mci=Corrected_error Error_enabled, mca=No Error, cpu_type= 		Intel generic architectural MCA, cpu= 0, socketid= 0, addr= abcd, mcgstatus=0, mcgcap= f000814, apicid= 0
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]: cpu 01:rasdaemon: mce_record store: 0x1d3ab48
		 May 16 15:23:24 vstx2-PowerEdge-R640 rasdaemon[23607]: rasdaemon: register inserted at db

		# ras-mc-ctl --errors
		  No Memory errors.
		  No PCIe AER errors.
		  No Extlog errors.
		  MCE events:
		  1 2024-05-16 10:21:24 -0700 error: No Error, mcg mcgstatus=0, mci Corrected_error Error_enabled, mcgcap=0x0f000814, 		  status=0x9400000000000000, addr=0x0000abcd, walltime=0x66464093, cpuid=0x00050654, bank=0x00000001
		  2 2024-05-16 10:21:24 -0700 error: No Error, mcg mcgstatus=0, mci Corrected_error Error_enabled, mcgcap=0x0f000814, 		  status=0x9400000000000000, addr=0x00001234, walltime=0x66464093, cpu=0x00000001, cpuid=0x00050654, apicid=0x00000020, 		socketid=0x00000001, bank=0x00000002

		# ras-mc-ctl --summary
		  No Memory errors.
		  No PCIe AER errors.
		  No Extlog errors.
		  MCE records summary:
			2 No Error errors

		# ras-mc-ctl --error-count
		  Label           CE      UE
		  Chan#0_DIMM#0   0       0
		  Chan#0_DIMM#1   0       0
		

# PCIeD Integration in SONiC 4.4.0

# Current Behaviors

PCIeD is running as daemon in the PMON container for all platforms. 

	root          90       1  0 Apr28 pts/0    00:02:51 python3 /usr/local/bin/pcied
## Work Proposed for SONiC 4.4.0

* Enhance TS to Collect PCIeD Outputs:

		root@sonic:/usr/bin# show platform pcieinfo     -> Show current device PCIe info
		root@sonic:/usr/bin# show platform pcieinfo -c  -> Check whether the PCIe info is correct
		
	Outputs:
		root@sonic:/home/admin# show platform pcieinfo
		===========================Display listed PCIe Device===========================
		bus:dev.fn 00:1f.2 - dev_id=0x8c02, SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 		6-port SATA Controller 1 [AHCI mode] (rev 05)
		bus:dev.fn 05:00.0 - dev_id=0x15ab, Ethernet controller: Intel Corporation Ethernet Connection X552 10 GbE 		Backplane
		bus:dev.fn 05:00.1 - dev_id=0x15ab, Ethernet controller: Intel Corporation Ethernet Connection X552 10 GbE 		Backplane
		bus:dev.fn 07:00.0 - dev_id=0xb873, Ethernet controller: Broadcom Inc. and subsidiaries Device b873 (rev 01)
		bus:dev.fn 0b:00.0 - dev_id=0x165f, Ethernet controller: Broadcom Inc. and subsidiaries NetXtreme BCM5720 2-port Gigabit Ethernet PCIe
		 
		root@sonic:/home/admin# show platform pcieinfo -c
		============================Check listed PCIe Device============================
		PCI Device:  SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 6-port SATA Controller 		1 [AHCI mode] (rev 05)  [Passed]
		PCI Device:  Ethernet controller: Intel Corporation Ethernet Connection X552 10 GbE Backplane ----------- 		[Passed]
		PCI Device:  Ethernet controller: Intel Corporation Ethernet Connection X552 10 GbE Backplane ----------- 		[Passed]
		PCI Device:  Ethernet controller: Broadcom Limited Device b873 (rev 01) --------------------------------- 		[Passed]
		PCI Device:  Ethernet controller: Broadcom Limited NetXtreme BCM5720 Gigabit Ethernet PCIe -------------- 		[Passed]
		PCIe Device Checking All Test ----------->>> PASSED
		

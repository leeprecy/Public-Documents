

# BMC Improvement Suggestions For SONiC AI Package

## High Level Design Document
**Rev 0.1**

## Table of Contents

* [Revision](#revision)
* [About This Manual](#about-this-manual)
* [Introduction](#Introduction)
* [Suggestions](#Suggestions)


# Revision

Rev   |   Date   |  Author   | Change Description
:---: | :-----:  | :------:  | :---------
0.1   | 06/2024 | Precy Lee | Initial version
        


# About this Manual

This document describes the BMC improvement suggestions for SONiC AI package. 

# Introduction

The BMC is an embedded microprocessor that sits on most switches and is responsbile for remote power management, sensros, serial console, and other feature such as virtual media. 


# Proposals

* When IPMI hardware-related event occurs, BMC sends event to notify SONiC to trigger custom action such as reboot. 

* SONiC serves as an event message source, sending Event messages to the SEL. This functionality enables SONiC to capture critical errors, ensuring essential information is available for post-mortem analysis if system shutdown, power-cycling, resetting, or other offline actions become necessary in response to a system event.  

* Automate SEL Management

  The BMC has a non-volatile memory buffer that stores up to 512 system events in a System Event Log (SEL). The SEL is stored in onboard flash memory on the BMC. The primary purpose of SEL is to help administrators diagnoses system issues. If the buffer is full, new messages are dropped. Therefore, the SEL policy is recommended to backup the SEL and clear the SEL after the backup operation occurs. The current SEL and backup SEL files can be collected to TS. 




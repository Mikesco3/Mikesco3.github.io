---
title: Migrating Production Servers to Virtual Instances Safely
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2022-11-05 09:22:00 -0500
categories: [Blogging,Planning]
tags: [hypervisors, vm, servers, migration, planning, p2v, ransomware]
# pin: false
---
# Virtualizing Physical Servers

## Why, What for...
This is intended to propose the strategic planning for a lot of situations where someone needs to migrate a critical production server, perhaps running on physical hardware or even another hypervisor, to a Virtual Instance running within a new hypervisor. 

For example: 
* If currently having licensing issues, or 
* The software is running on hardware that is getting too old, and it may not be possible to find hardware capable of supporting it. 
* To protect the server against a potential ransomware situation or many other catastrophic failures.
* To gain the advantages of high-availability, live migration and cross-replication to other physical hardware.

*In this example we are going to consider someone who is running a physical server that is getting too old, and migrating this server with all of its software and data, to running within a virtual machine within a hypervisor (such as XCP-ng, Proxmox, VmWare, etc.)*

*We are also not going to go into the technical steps involved in this process (as this is going to be specific to the preferred hypervisor, backup software, etc.) but first give the high-level plan for a strategy that would be the applicable to a wide choice of hypervisors.* 

*The specific steps can be documented and detailed once the finer details about the solution have been established.*

## Important Considerations
It is important that at every step:
* Make a good backup before you even begin.
* Work on a copy, not on the production environment.
* Give yourself the ability to fall back on a working solution.
  (*don't discontinue the old working server until the new one has proved capable of handling the situation even under high stress*)
* Additionally, make sure:
  - To create a resiliency / disaster recovery plan going forward. 
  - The new instance is being properly backed up.
  - The new instance is not the only working version.
  - There is an underlying snapshotting system taking periodic snapshots of the live server at optimal intervals (15 minutes, hourly, daily, weekly, monthly...)
  - It can be replicated to a secondary shadow instance that could take over if the main one crashes. 
  - It would be even better if the snapshotting system underneath is being replicated to a secondary physical server, even at a different physical location. (aside of a high-availability cluster).
  - There could be a high-availability cluster of 3 or more physical servers underneath it.

## The Process
### Most likely Scenario
1. Grab a full Image backup of the working instance of the physical server. 
2. Prepare the Hypervisor that will run the Virtual Instance.
   *This doesn't even need to be the eventual server where the production instance will run on.*
3. Restore the backup into a Virtual Machine. 
4. Optimize the Virtual Machine 
   *Remove old unneeded software, drivers, temporary data.*
5. Bring on a user that has experience running the production version to test the new virtual instance. ^14ea08
6. Set a time and date for a crossover transition, where there is no risk for data to be accidentally duplicated or split by being modified on both instances. ^760e81
7. Transfer the Data from the Physical Instance that has been up to this point the production server to the new Virtual Instance. (This is just the database or customer files)
8. Swap the Servers (either by renaming them, detaching/attaching from the network or by powering the old one off and the new one on to take its place.)
9. Test again the New Virtual Instance under load.
10. Make sure the high-availability / Snapshotting / Replication and Backup processes are already functioning. 
11. Keep the Old Server on standby ready to take the place of the new one in case of any unpredictable outcomes. *Determine a safe period to keep the old one around on standby for*.
12. Document and establish periodic Maintenance / Checkup routine. 
   
This process assumes that there is an opportunity to migrate the remaining live production data, that would have changed between the initial full image backup, (between **step 1** and **step 7**) until the time of bringing up the new Virtual Instance.

### Alternate Scenario
If there isn't a chance to transfer just the production data that changed separately after the full system backup, then the process would be almost identical to the one for the most likely scenario, except that:
* The first time around you would run the whole process on a dry-run test (**step 1** through **step 6** of the Most likely scenario would still remain the same).
* Now right before **step 7** of the most likely scenario, the process would involve:
7. Document the whole transition of **step 3** through **step 5** where you convert the physical server to a viable Virtual Instance and have had someone test it.
8. Simulate a short production run with a lot more staff running it as hard as possible.
9. You may do one more dry-run of the conversion from the physical to the virtual instance, to test the documented process once more, to get an idea whether the process is repeatable and to obtain a more realistic idea of the new time-frame for the conversion, now that you are following a known procedure.
10. Set the high-availability, snapshotting, replication and backup in place.
11. Find, schedule and communicate a time to halt all production for the actual migration.
12. Halt all activity on the Production Server.
13. Migrate the Physical Server to the new Virtual Instance.
*From here on, **step 10** through **step 12** of the Most Likely Scenario should be more or less the same.*
---
layout: post
title: Improved Performance by Hosting Virtual Hard Disk External USB Drives 
category: Virtualization, Hyper-V
---

I think I already knew this to be true, but didn't own a "portable enough" hard disk to lug around with my laptop to try it out myself.  That's about to change as I got myself new Western Digital My Passport Ultra today; its the perfect size both terms of capacity and dimensions. So now I get to try this out in a real world scenario.

### Moving the Virtual Drive Images 

The entire process entails moving the physical files to the new location and letting Hyper-V know about this move. If the virtual machine (VM) is already active it does not seem to be possible to move the checkpoint location.


1. Open up the Hyper-V Management Console.

	![Hyper-V Management Console](/images/posts/VirtualHardDiskPerf/1_HyperVManager.png)
2. Create a checkpoint of the VM.
3. Shutdown the VM.
4. Next, navigate to the VM setting panel by right-clicking and selecting `Settings` or select it from the right actions pane.
5. On the settings panel, navigate to the `hardware > IDE Controller 0`
6. `Virtual hard disk` text box shows the current location of the virtual drive.

	![Virtual Machine Settings](/images/posts/VirtualHardDiskPerf/2_HyperVSettings.png)
7. Copy the contents of this entire directory together with the snapshots to the new destination.

	![Copy Virtual Machine Files](/images/posts/VirtualHardDiskPerf/9_FolderLocation.png)
8. Change the `Virtual hard disk` path to match the new location.

	![Updated Virtual Machine Settings](/images/posts/VirtualHardDiskPerf/3_UpdatedHyperVSettings.png)
9. You would receive the following message warning of data loss:

	![Data Loss Warning](/images/posts/VirtualHardDiskPerf/4_DataLossWarning.png)
10. Since a checkpoint was created initially, click on `Continue`.
11. Click on `Ok` and the following warning is shown. Click on `Continue` as the warning not applicable to us.
	
	![Chain Breakage](/images/posts/VirtualHardDiskPerf/5_DataLossWarning2.png)
12. Start up the virtual machine.

Since I already have checkpoint snapshots of my VMs, Hyper-V did not allow me to change the location. I think I might be able to use symbolic links in order to trick the OS into using the new drive. I will try to explore techniques available in a future post.

![Checkpoint Location](/images/posts/VirtualHardDiskPerf/6_CheckPointLocation.png)

The only time this has become an issue is when saving the state of the VM. This causes hyper-v to writes a significant amount to the primary drive. This is something I can live with for now. 

### Benchmarks

I decided to do a quick benchmark on the hard disk to satisfy my curiosity. I used CrystalDiskMark as it appears to be one of the more popular ones.

![My Passport Ultra CrystalDiskMark](/images/posts/VirtualHardDiskPerf/7_WD_CrystalDiskMark.png)

The numbers are on par with what's to be expected of the drive - here's a 33 gig VPC being copied over.  

![My Passport Ultra CrystalDiskMark](/images/posts/VirtualHardDiskPerf/8_WD_BasicFileCopy.png)

### Conclusion

While I do not have a conclusive way to prove that get I do get improved performance, my system does feels more responsive. My primary drive no longer chokes with 100% activity when working with multiple VMs.

  

### Future Improvements

Setting up the operating system on a Solid State Drive (SSD) should be the next step in improving the overall system performance. Replacing the slow mechanical hard drive should in theory bring all kinds of performance improvements. And in order to get the maximum benefit of SSDs, one would need to think about strategies such as partitioning schemes in effectively segmenting data. 

### References

- [Manually Merging Hyper-V Checkpoints](https://workinghardinit.wordpress.com/tag/avhdx/)
- [CrystalDiskMark](http://crystalmark.info/software/CrystalDiskMark/index-e.html)
- [Using Symbolic Links to Save Space on Your SSD](http://blog.danieljost.com/symbolic-links-save-space-ssd/)

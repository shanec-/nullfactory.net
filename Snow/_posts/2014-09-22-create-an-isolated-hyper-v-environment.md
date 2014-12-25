---
layout: post
title: Create An Isolated Hyper-V Environment
category: Virtualization, Hyper-V
---
*Bad things may happen when you power up a virtualized Domain Controller on your laptop and connect it to the corporate network.*

This post focuses on building a self-contained, isolated virtual environment with internet connectivity.

My colleague, Chaminda has a [detailed post](http://chamindac.blogspot.com/2013/12/setup-virtual-environment-for-tfs-2013_19.html) on how to setup and isolated environment using virtual box. Go check it out if you would like to implement it via virtual box.  While virtual box is a good virtualization platform on its own right, I have grown accustomed to using Hyper-V in my day-job and has become a personal preference.

## Hyper-V

My own environment is built around this [excellent post](http://blogs.technet.com/b/askpfeplat/archive/2013/03/04/your-personal-isolated-lab-featuring-windows-8-hyper-v.aspx). It details the entire process involved. While my own setup is identical to the above, I have taken into account the following caveats:

1. As mentioned by one of the comments on the post, it is important to explicitly set the port to `eth0` soon after flashing the image.
2. Port Forwarding - Even simple tasks like setting up share folders require that certain ports be accessible. Therefore this it is an important consideration when planning an isolated environment. Here are some of the services and ports I've used for my TFS environment:

	![Port Forwarding](/images/posts/IsolatedEnvironment/1_PortForwarding.png)

### Setting up Routing

Even though I have my isolated environment, there are instances where I would like resource in my main network to have access to the internal network. Although port forwarding works to a certain degree, we run into its limitations very fast.

The ideal
This involves setting up routes on both out internal router as well as the external router under which the external resources exists.

## References

- [Your Personal Isolated Lab - Featuring Windows 8 + Hyper-V](http://blogs.technet.com/b/askpfeplat/archive/2013/03/04/your-personal-isolated-lab-featuring-windows-8-hyper-v.aspx)
- [Create a PDC in an Isolated Internal Network - Setup Virtual Environment for TFS 2013 - Using Virtualbox](http://chamindac.blogspot.com/2013/12/setup-virtual-environment-for-tfs-2013_19.html)
- [Understanding Shared Folders and the Windows Firewall](http://technet.microsoft.com/en-us/library/cc731402.aspx)

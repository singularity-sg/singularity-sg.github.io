---
layout: post
title: "Living a Vagrant life"
description: "Keeping your project development environment clean and reproducible"
category: "Software Development Practices"
tags: ['vagrant','virtualization','development environment' ]
---
{% include JB/setup %}

Anyone that had been working on multiple projects on the same boxes will know that there is never a certainty if the development environment you have worked on for a particular project will still be able to run months down the road. It'll probably be filled with all sorts of gunk and remnants of other similar projects which would cause problems when you try to run your code again. This problem was especially pertinent if you run code that is very environment sensitive e.g. Ruby on Rails and its gems.

Thus, the idea of an automatically provisioned development environment that can be spun up instantly in a clean state was something that had been talked about for a long time in my agile development team. Recently, we came across Vagrant (http://www.vagrantup.com/) and it seemed like the panacea to our coding environment problems. I had it setup for a Ruby On Rails project and it will automatically install RVM, Ruby gems and all the native packages for the Vagrant box. It will also automatically provision MySQL and even installs an Aptana Studio ide.

I installed Vagrant on 4 powerful development machines (a mix of Ubuntu Linux, CentOS Linux and Windows 7 host machines). Each of these machines contain an 8-core Xeon processor and 16 GB of RAM and I deemed that sufficient to startup Virtual Machines without problems. At least, that was what I think.

When we started to run Vagrant with VirtualBox, the performance of the software within the guest OS was appallingly slow. After some googling for more information about this apparent deficiency, I found many articles describing VirtualBox's slow filesystem read operations if you are accessing a shared folder. This is exactly what Vagrant needs to do in order for it to access your source code. This article explains in detail the various performance for Vagrant using VirtualBox vs VMWare (http://mitchellh.com/comparing-filesystem-performance-in-virtual-machines)

So, in the end, I had to configure Vagrant to access the shared folders via NFS for Linux-based Host machines. As for the Windows Host boxes, we had no alternatives and so running Vagrant on VirtualBox produces the kind of performance that would make your Grandmother feel like Usain Bolt.

I'm not sure if running Vagrant with VMWare would resolve some of the pain points of Vagrant but it seems you need to pay for a license to integrate Vagrant with VMWare. According to the article that compares the performance, it seemed that VMWare read operations are much faster than VirtualBox and so should provide a better performance.

I guess if one is serious about using Vagrant, it would be best to purchase the VMWare provider license and pair it with a Linux Host OS for maximum efficacy.

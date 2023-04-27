---
layout: post
title: "Introducing Sysdiagnose: A Comprehensive Tool for macOS System Data Collection"
date: 2023-04-27 17:00:00 +0300
categories: Security
---

Welcome to Justicial's guide on using Sysdiagnose, a powerful built-in macOS tool that will save you time and effort when collecting valuable system information.

Sysdiagnose gathers system diagnostic information helpful in investigating system performance issues. A great deal of information is harvested, spanning system state and configuration. The data is stored in the /var/tmp directory. Sysdiagnose needs to be run as root. To cancel an in-flight sysdiagnose triggered via the command-line interface, press Ctrl-\\.

If you have physical access to the machine, you can easily trigger Sysdiagnose by pressing Control-Option-Command-Shift-Period. This key combination automatically triggers Sysdiagnose, making it a convenient option for quickly collecting system data.

Alternatively, you can use the Terminal app and enter the following command:

{% highlight sh %}
sudo sysdiagnose
{% endhighlight %}

After providing the admin password and confirming, the process will start without the display flash. Sysdiagnose takes a few minutes to complete, allowing you to take a break or multitask while it works.

### What Sysdiagnose Collects

Sysdiagnose collects a wealth of information, including:

- A spindump of the system
- Several seconds of fs_usage output
- Several seconds of top output
- Data about kernel zones
- Status of loaded kernel extensions
- Resident memory usage of user processes
- Recent system logs
- A System Profiler report
- Recent crash reports
- Disk usage information
- I/O Kit registry information
- Network status
- If a specific process is supplied as an argument, it will collect:
  - A list of malloc-allocated buffers in the process's heap
  - Data about unreferenced malloc buffers in the process's memory
  - Data about the virtual memory regions allocated in the process

Once Sysdiagnose finishes, a Finder window will display the compressed results. Unpack the data and explore its contents to gain valuable insights into the system's performance and configuration.

To access Sysdiagnose logs, navigate to the logs folder and expand it. You'll find valuable files like Install.log and InstallHistory.plist, among others.
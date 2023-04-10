---
layout: post
title: "User macOS Security: Understanding Apple's Built-in Tools"
date: 2023-04-02
categories: Security
---
macOS has a reputation for being a secure operating system, but that doesn't mean it's immune to threats. Malware and other attack vectors can still affect macOS users, which is why it's important to understand the security tools and features that Apple has built into the operating system. In this post, we'll take a closer look at some of the built-in security tools that individual macOS users can access. While these tools can't guarantee absolute security, understanding how they work can help users make more informed decisions about how to protect themselves and their data.

## XProtect and MRT

Apple's static signature check (XProtect), and malware removal tool (MRT) are two built-in security features on macOS. These tools are located at `/Library/Apple/System/Library/CoreServices`. In general, you can find them using
{% highlight sh %}
mdfind -name XProtect | grep CoreServices
{% endhighlight %}
and
{% highlight sh %}
mdfind -name MRT | grep CoreServices
{% endhighlight %}
respectively.

**XProtect** is a technology that scans executable files for known malware families. It is a static signature check that must be written and pushed to your device by Apple. However, it cannot protect users against any malware that Apple has not already seen and developed a specific detection for.

On macOS versions prior to Catalina, malware typically bypassed XProtect by removing the quarantine bit. Although Apple now subjects all files to an XProtect scan regardless of the quarantine bit, that hasn’t prevented malware authors from finding bypasses.

Backing up XProtect is another tool that Apple uses for remediation purposes: the **MRT** (Malware Removal Tool). MRT primarily searches for known malware in certain locations (at certain file paths) and silently removes anything it matches.

To check a suspicious program, create a launchd plist in the `LaunchAgents` or `LaunchDaemons` directory and specify the path to it in the `Program` or `ProgramArguments` value of the plist. After that, open the Terminal and use the command below to run MRT:

{% highlight sh %}
/Library/Apple/System/Library/CoreServices/MRT.app/Contents/MacOS/MRT -a
{% endhighlight %}

For root level LaunchAgents and LaunchDaemons, use the command below instead:

{% highlight sh %}
sudo /Library/Apple/System/Library/CoreServices/MRT.app/Contents/MacOS/MRT -d
{% endhighlight %}

If MRT detects malware, it will output `Found infection`.

It's worth noting that analyzing the internal rules of MRT has become more complex as Apple has started to obfuscate strings within its binary. Additionally, Apple introduced a new tool called **XProtect.app**, located at `/Library/Apple/System/Library/CoreServices`, which serves as a substitute for MRT. XProtect.app includes separate binaries for well-known malware families and receives frequent updates. However, it's important to mention that XProtect.app is not compatible with older macOS versions.

## TCC (Transparency, Consent & Control)

**TCC** is a feature meant to protect user data from being accessed. In recent years, protecting sensitive user data on-device has become of increasing importance, particularly now that our phones, tablets, and computers are used for creating, storing, and transmitting.

With macOS, Apple took a strong position on protecting user data early on, implementing controls as far back as 2012 in OSX Mountain Lion under a framework known as ‘Transparency, Consent and Control’, or TCC for short.

Get more insights on the TCC framework used to protect user data on macOS in our related post: [Transparency, Consent, and Control (TCC) Technology on macOS]({{ site.baseurl }}{% link _posts/2023-04-09-transparency-consent-and-control-tcc-technology-on-macos.md %}).

**In conclusion**, macOS has a set of built-in security tools that can help users protect their system from malicious software. By understanding how these tools work, users can take proactive measures to secure their device and prevent potential threats. From Gatekeeper (which you can learn more about in the post [Understanding Gatekeeper: Apple's Security Technology for macOS]({{ site.baseurl }}{% link _posts/2023-03-31-understanding-gatekeeper-apples-security-technology-for-macos.md %})) to MRT and XProtect, these tools are designed to work together to create a secure environment for macOS users. However, it's important to keep in mind that no tool or system is completely foolproof, and it's always a good idea to stay informed about the latest security threats and best practices. By staying vigilant and utilizing the tools available, macOS users can minimize their risk of falling victim to malware and other security threats.
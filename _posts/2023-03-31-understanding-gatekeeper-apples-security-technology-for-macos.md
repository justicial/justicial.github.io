---
layout: post
title: "Understanding Gatekeeper: Apple's macOS Security Technology"
date: 2023-03-31 10:00:00 +0300
categories: Security
---

Gatekeeper is an essential security technology in macOS, designed to ensure that only trusted software runs on a user's Mac. It verifies the software's authenticity by checking that it comes from an identified developer, has been verified and notarized by Apple to be free from known malicious content, and remains unaltered. In this article, we will discuss how Gatekeeper operates, its development over time, and its crucial features as a fundamental component of the macOS security framework.

By default, Gatekeeper verifies that all downloaded software has been signed by the App Store or a registered developer and notarized by Apple. It checks software for known malicious content when first opened, regardless of the source. Users and organizations can choose to allow only App Store software or override Gatekeeper policies as needed, and mobile device management (MDM) solutions can be used to configure settings. Additionally, Gatekeeper safeguards against malicious plug-ins distributed with benign apps by opening them from randomized, read-only locations when necessary.

## Historical Reference

The evolution of Gatekeeper can be traced back to Mac OS X 10.4 Tiger’s `Download Validation`, which was introduced to protect users from potentially malicious content by warning them about unsafe file types.

This feature was further improved upon in Mac OS X 10.5 Leopard, which introduced `File Quarantine`. With this feature, macOS remembers which content the user obtained from a network, and the first time a potentially unsafe file is opened in Finder, Spotlight, or from the Dock, the file quarantine feature will warn the user about unsafe file types. The user is advised to cancel if they have any doubts about the file.

Gatekeeper was added to the operating system starting from Mac OS X 10.7.3 Lion in a preview state and enabled by default for users in Mountain Lion 10.8. Starting from macOS 13, Gatekeeper can handle anti-tampering for notarized apps.

## Gatekeeper and User Control

Gatekeeper provides three levels of security for macOS users: `App Store`, `App Store and identified developers`, and `Anywhere`. The `App Store` option only allows apps that have been downloaded from the Mac App Store to run, ensuring that the apps come from verified developers and have been signed with a valid Apple Developer ID. The `App Store and identified developers` option allows apps downloaded from the App Store and those signed by identified developers to run. Finally, the `Anywhere` option allows all applications to run, regardless of where they were downloaded.

The Gatekeeper interface can be accessed by navigating to `System Preferences` > `Security & Privacy` > `General`, and selecting one of the three options for `Allow apps downloaded from`. Note that the `Anywhere` option is hidden by default starting from macOS Sierra (version 10.12). However, it can be enabled by disabling Gatekeeper from the command line. After that, the `Anywhere` option will be shown in the `Security and Privacy` panel. Users can change the setting as needed depending on their requirements, but it is recommended to keep the Gatekeeper setting on the App Store and identified developers option.

While Gatekeeper is an essential security feature for macOS users, it can be disabled using the following command in Terminal:
{% highlight sh %}
sudo spctl --master-disable
{% endhighlight %}

This command disables Gatekeeper, allowing any app to run on the system, including those that have not been signed by a developer. Disabling Gatekeeper can leave the system vulnerable to potentially harmful software, so it is recommended that users only disable it when absolutely necessary and re-enable it as soon as possible. To re-enable Gatekeeper, use the following command in Terminal:
{% highlight sh %}
sudo spctl --master-enable
{% endhighlight %}

It is important to exercise caution when opening any downloaded files, especially if they are not from a trusted source. Gatekeeper works in conjunction with other macOS security features, such as Quarantine Extended Attributes and the QuarantineEvents Database. These features help in identifying and flagging potentially unsafe files that have been downloaded from the internet or created by certain applications. Understanding the details of these features is essential for a more comprehensive view of macOS security.

For more information on macOS Quarantine Extended Attributes and the QuarantineEvents Database, as well as how they interact with Gatekeeper, please refer to our dedicated article [Understanding macOS Quarantine Extended Attributes and QuarantineEvents Database]({{ site.baseurl }}{% link _posts/2023-04-20-understanding-macos-quarantine-extended-attributes-and-quarantineevents-database.md %}). This article delves deeper into these topics, providing valuable insights and examples that will help you better understand the nuances of macOS security.

## System Policy Database

The System Policy database is located at `/var/db/SystemPolicy` and includes four tables: `authority`, `bookmarkhints`, `feature`, and `object`. The authority table is a ledger consisting of an allow/deny decision, the assessed code-signing requirements’ string or cdhash, and the responsible assessor. Most entries have the GKE (Gatekeeper’s exclusions) as the assessor. A SIP-protected (plist) listing enumeration of these exclusions is found at `/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`. When new apps are assessed by Gatekeeper, this will be reflected in the authority table. The System Policy database is not protected by SIP.

## Other DBs

In addition to the databases mentioned above, Gatekeeper actively uses the following databases:
* /var/db/SystemPolicyConfiguration/ExecPolicy
* /var/db/SystemPolicyConfiguration/KextPolicy
* /var/db/SystemPolicyConfiguration/Tickets

These databases are used by the system to determine whether a certain executable, kernel extension, or code signature should be trusted. The `ExecPolicy` database is used for allowing or denying the execution of specific binaries or scripts. The `KextPolicy` database is used for allowing or denying the loading of specific kernel extensions. The `Tickets` database is used for managing software update tickets. These databases are essential to the functioning of Gatekeeper, and they are protected by SIP, which makes them very difficult for adversaries to tamper with.

## To sum up

In conclusion, Gatekeeper is an essential security feature for macOS users that protects them from potentially harmful content. It evolved from an opt-in feature called File Quarantine and has become an integral part of the macOS security framework. Gatekeeper checks for quarantine flags and notarization tickets to ensure the validity of applications and files. By providing three levels of security, users can choose the option that best suits their needs while maintaining the highest level of security. However, it's important to note that Gatekeeper is not bulletproof and that bypass techniques have been found in the past, so users should always exercise caution when opening any downloaded files, especially if they are not from a trusted source.
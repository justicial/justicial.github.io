---
layout: post
title: "Understanding Gatekeeper: Apple's Security Technology for macOS"
date: 2023-03-31 10:00:00 +0300
categories: Security
---
Gatekeeper is a significant security technology developed by Apple to protect macOS users from potentially harmful software. Apple is well known for being picky about what users can install on their operating systems. In this article, we will explain how Gatekeeper works, its evolution, and the features that make it an integral part of the macOS security framework.

## Historical Reference

The evolution of Gatekeeper can be traced back to Mac OS X 10.4 Tiger’s `Download Validation` that was introduced to protect users from potentially malicious content by warning them about unsafe file types.

This feature was further improved upon in Mac OS X 10.5 Leopard, which introduced `File Quarantine`. With this feature, macOS remembers which content the user obtained from a network, and the first time a potentially unsafe file is opened in Finder, Spotlight, or from the Dock, the file quarantine feature will warn the user about unsafe file types. The user is advised to cancel if they have any doubts about the file.

Gatekeeper was added to the operating system starting from Mac OS X 10.7.3 Lion in a preview state and enabled by default for users in Mountain Lion 10.8. Starting from macOS 13, Gatekeeper can handle anti-tampering for notarized apps.

## Gatekeeper and User Control

Gatekeeper provides three levels of security for macOS users: `App Store`, `App Store and identified developers`, and `Anywhere`. The `App Store` option only allows apps that have been downloaded from the Mac App Store to run, ensuring that the apps are from verified developers and have been signed with a valid Apple Developer ID. The `App Store and identified developers` option allows apps downloaded from the App Store and those signed by identified developers to run. Finally, the `Anywhere` option allows all applications to run, regardless of where they were downloaded.

The Gatekeeper interface can be accessed by navigating to `System Preferences > Security & Privacy > General`, and selecting one of the three options for "Allow apps downloaded from." Users can change the setting as needed depending on their requirements, but it is recommended to keep the Gatekeeper setting on the "App Store and identified developers" option.

## Quarantine Extended Attribute

When documents, applications, or programs are downloaded, an extended attribute called `com.apple.quarantine` can be set on the file by the application performing the download. It can also be set to files the application creates (if the application opts-in to this feature by setting `LSFileQuarantineEnabled` to `YES` in its Info.plist). The presence of this quarantine flag initiates a Gatekeeper check. If the quarantine flag is set in macOS 10.15 or later, Gatekeeper also checks for a notarization ticket and sends a cryptographic hash to Apple's servers to check for its validity.

Apps and files loaded onto the system from a USB flash drive, optical disk, external hard drive, from a drive shared over the local network, or using the `curl` command do not get this flag.

The quarantine flag, also known as a quarantine extended attribute, is among the stickiest of all xattrs. When an archive that has been flagged is unzipped, the xattr is normally propagated to all items that are saved from it. This ensures that compressed apps retain their flag when uncompressed.

The presence of the `com.apple.quarantine` flag can be checked with the `/usr/bin/xattr` utility:
{% highlight sh %}
xattr -l /path/to/file
{% endhighlight %}
or
{% highlight sh %}
xattr -p com.apple.quarantine /path/to/file
{% endhighlight %}

In macOS Mojave and later, a typical quarantine xattr consists of a Unicode string of the form:
{% highlight xattr %}
0003;6405dcc3;Safari;7F8A8A9D-4B68-4AB3-B4C7-71468256EF72
{% endhighlight %}

The components of this string are:
* The quarantine value in hexadecimal
* The time in which the file was downloaded, Unix hexadecimal timestamp
* The name of the application that downloaded the file
* A UUID identifying the download

It can also be recursively removed:
{% highlight sh %}
xattr -dr com.apple.quarantine /path/to/file
{% endhighlight %}

It is important to note that removing the quarantine flag does not guarantee that the file is safe, and users should exercise caution when opening any downloaded files, especially if they are not from a trusted source.

## QuarantineEvents Database

The QuarantineEvents database is an SQLite database located at `~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`. It includes only one table named `LSQuarantineEvent` with the following columns:
{% highlight sql %}
LSQuarantineEventIdentifier (TEXT PRIMARY KEY NOT NULL)
LSQuarantineTimeStamp (REAL)
LSQuarantineAgentBundleIdentifier (TEXT)
LSQuarantineAgentName (TEXT)
LSQuarantineDataURLString (TEXT)
LSQuarantineSenderName (TEXT)
LSQuarantineSenderAddress (TEXT)
LSQuarantineTypeNumber (INTEGER)
LSQuarantineOriginTitle (TEXT)
LSQuarantineOriginURLString (TEXT)
LSQuarantineOriginAlias (BLOB)
{% endhighlight %}

Due to the QuarantineEvents database not being protected by SIP, it has been widely used by adversaries to read their campaign identifiers. However, as of macOS 12.4, Apple has stopped adding URLString to this table. As a result, we can see interesting cases of the new methods of storing such kind of configuration, for instance, it can be [DMG's internal structure](https://blog.confiant.com/lart-de-l-%C3%A9vasion-how-shlayer-hides-its-configuration-inside-apple-proprietary-dmg-files-73586b6e7f8d) as Taha Karim pointed out.

## System Policy Database

The System Policy database is located at `/var/db/SystemPolicy` and includes four tables: `authority, bookmarkhints, feature, and object`. The authority table is a ledger consisting of an allow/deny decision, the assessed code-signing requirements’ string or cdhash, and the responsible assessor. Most entries have the GKE (Gatekeeper’s exclusions) as the assessor. A SIP-protected (plist) listing enumeration of these exclusions is found at `/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`. When new apps are assessed by Gatekeeper, that will be reflected in the authority table. The System Policy database is not protected by SIP.

## Other DBs

Besides the mentioned above databases, Gatekeeper actively uses the following databases:
* /var/db/SystemPolicyConfiguration/ExecPolicy
* /var/db/SystemPolicyConfiguration/KextPolicy
* /var/db/SystemPolicyConfiguration/Tickets

These databases are used by the system to determine whether a certain executable, kernel extension or code signature should be trusted. The ExecPolicy database is used for allowing or denying the execution of specific binaries or scripts. The KextPolicy database is used for allowing or denying the loading of specific kernel extensions. The Tickets database is used for managing software update tickets. These databases are essential to the functioning of Gatekeeper, and they are protected by SIP, which makes them very difficult for adversaries to tamper with.

## To sum up

In conclusion, Gatekeeper is an essential security feature for macOS users that protects them from potentially harmful content. It evolved from an opt-in feature called File Quarantine and has become an integral part of the macOS security framework. Gatekeeper checks for quarantine flags and notarization tickets to ensure the validity of applications and files. By providing three levels of security, users can choose the option that best suits their needs while maintaining the highest level of security. However, it's important to note that Gatekeeper is not bulletproof and that bypass techniques have been found in the past, so users should always exercise caution when opening any downloaded files, especially if they are not from a trusted source.
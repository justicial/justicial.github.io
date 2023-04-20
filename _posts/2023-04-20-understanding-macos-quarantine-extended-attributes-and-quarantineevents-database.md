---
layout: post
title: "Understanding macOS Quarantine Extended Attributes and QuarantineEvents Database"
date: 2023-04-20 14:00:00 +0300
categories: Security
---

## Demystifying the Quarantine Extended Attribute in macOS

When users download files, applications, or programs on macOS, an extended attribute called `com.apple.quarantine` is often assigned to the file by the downloading application. This attribute can also be applied to files created by the application if `LSFileQuarantineEnabled` is set to YES in its Info.plist. The existence of the quarantine flag prompts a Gatekeeper verification. Additionally, in macOS 10.15 and later, Gatekeeper checks for a notarization ticket and sends a cryptographic hash to Apple's servers to confirm its validity.

Files transferred to the system from external storage devices, local network drives, or using the `curl` command do not receive this flag.

The quarantine flag, or quarantine extended attribute, is one of the most persistent xattrs. When an archive with this flag is decompressed, the xattr is generally spread to all items extracted from it, ensuring that uncompressed apps maintain their flag.

You can use the `/usr/bin/xattr utility` to check for the presence of the `com.apple.quarantine` flag:
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
* The file download time as a Unix hexadecimal timestamp
* The name of the application that downloaded the file
* A UUID identifying the download

To remove the quarantine flag recursively, use:
{% highlight sh %}
xattr -dr com.apple.quarantine /path/to/file
{% endhighlight %}

Keep in mind that removing the quarantine flag does not guarantee the file's safety. Users should always be cautious when opening downloaded files, especially if they come from untrusted sources.

## Reading the Quarantine Attribute with NSURL Class

Another way to access the quarantine attribute is by utilizing the `resourceValuesForKeys:error:` method of the `NSURL` class. For example, the following code snippet shows how to print a file's quarantine attribute:
{% highlight Objective-C %}
NSURL *url = [NSURL fileURLWithPath:@"/path/to/file"];
NSDictionary *values = [url resourceValuesForKeys:@[NSURLQuarantinePropertiesKey] error:nil];
NSLog(@"%@", values[NSURLQuarantinePropertiesKey]);
{% endhighlight %}

The output may resemble:
{% highlight log %}
{
LSQuarantineAgentBundleIdentifier = "com.google.Chrome";
LSQuarantineAgentName = Chrome;
LSQuarantineEventIdentifier = "EF85529B-485D-43E3-9DE2-9533D6EF8C79";
LSQuarantineIsOwnedByCurrentUser = 1;
LSQuarantineTimeStamp = "2023-04-20 11:13:28 +0000";
LSQuarantineType = LSQuarantineTypeWebDownload;
}
{% endhighlight %}

This method provides an alternative approach to obtaining the quarantine attribute of a file, offering developers and users more flexibility when working with macOS security features. It is important to be familiar with various methods of accessing and manipulating quarantine attributes, as doing so can contribute to a more secure user experience and help prevent potential security risks.

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

The QuarantineEvents database is not protected by SIP, so adversaries have frequently used it to read their campaign identifiers. However, starting with macOS 12.4, Apple no longer adds URLString to this table. As a result, new methods for storing such configuration information have emerged, such as [using DMG's internal structure](https://blog.confiant.com/lart-de-l-%C3%A9vasion-how-shlayer-hides-its-configuration-inside-apple-proprietary-dmg-files-73586b6e7f8d), as highlighted by Taha Karim.

## Understanding the Implications of Quarantine Attributes and QuarantineEvents

The quarantine extended attribute and the QuarantineEvents database play crucial roles in macOS security. They ensure that files downloaded from the internet or created by applications are verified and safe before being opened by the user.

However, due to the lack of SIP protection on the QuarantineEvents database, adversaries have been able to exploit it for their campaigns. While Apple has taken steps to mitigate this by no longer adding URLString to the table, it is essential for users to stay vigilant and informed about new tactics that attackers may employ.

In conclusion, the macOS quarantine system is an essential security measure designed to protect users from potentially harmful files. It is crucial for users to understand how this system works, exercise caution when opening files from untrusted sources, and stay up-to-date on the latest security measures to minimize the risks associated with downloading and opening files on their devices.
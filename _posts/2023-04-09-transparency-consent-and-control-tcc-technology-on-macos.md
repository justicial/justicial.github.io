---
layout: post
title: "Transparency, Consent, and Control (TCC) Technology on macOS"
date: 2023-04-09
categories: security
---

**Transparency, Consent, and Control (TCC)** technology is a critical component of the macOS privacy framework. It was first introduced by Apple in 2012 on macOS Mountain Lion, and its primary function is to provide users with the ability to configure privacy settings for their apps. This includes access to the device’s camera, microphone, or location, as well as access to the user’s calendar or iCloud account, among others.

One of the key features of TCC is its ability to prevent apps from accessing users’ personal information without their prior consent and knowledge. Users can manage TCC under System Preferences in macOS (`System Preferences > Security & Privacy > Privacy`). TCC maintains databases that contain consent history for app requests.

When an app requests access to protected user data, one of two things can happen:

- If an app has previously requested a specific permission, its status is marked as either granted or not, and the system will not prompt the user again. To reset this flag, the user must use the `tccutil` utility, which I will discuss below.
- When an app requests a specific permission for the first time, the system will present a prompt asking the user to grant or deny access. A new record for that app will be added to the TCC database, regardless of the user's choice. Subsequent requests from that app for the same permission will then fall under the first scenario, and the user will not be prompted again.

Under the hood, there are two kinds of TCC databases. Each type only maintains a subset of the request types:

1. **User-specific database**: contains stored permission types that only apply to the specific user profile; it is saved under `~/Library/Application Support/com.apple.TCC/TCC.db` and can be accessed by the user who owns the profile.
2. **System-wide database**: contains stored permission types that apply on a system level; it is saved under `/Library/Application Support/com.apple.TCC/TCC.db` and can be accessed by users with root or full disk access.

However, the TCC.db file is a SQLite database, and if a user is granted full disk access, they can view and even edit the database. Having full disk access to the TCC databases can allow malicious actors to edit them and grant arbitrary permissions to any app they choose, including their own malicious app. The affected user would also not be prompted to allow or deny those permissions, thus allowing the app to run with configurations they may not have known or consented to.

To address this issue, Apple made two changes. First, Apple protected the system-wide TCC.db via System Integrity Protection (SIP), a macOS feature that prevents unauthorized code execution. Secondly, Apple enforced a TCC policy that only apps with full disk access can access the TCC.db files.

In addition, macOS implements the TCC logic by using a special daemon called **tccd**. Indeed, there are at least two instances of `tccd`: one run by the user and the other by root. All permissions are managed by the TCC.framework at `/System/Library/PrivateFrameworks/TCC.framework`. The TCC.framework is a private framework, and it is not possible to access it directly from user space. Instead, the TCC.framework is accessed via the `tccutil` command-line tool, which is a wrapper around the TCC.framework. `/usr/bin/tccutil` claims to offer the ability “to manage the privacy database”, but the only documented command is `reset`. This command can reset the TCC database for either all applications or a specified application bundle identifier. Here are some examples of how to use the `tccutil` command:

{% highlight sh %}
# resets the entire TCC database for all applications
tccutil reset All

# resets the TCC database for the Camera service for the app with the bundle identifier `com.example.app`
tccutil reset Camera com.example.app
{% endhighlight %}

Finally, it is worth noting that there is one app on the Mac that always has `Full Disk Access` but never appears in the `Full Disk Access` pane in System Preferences: the `Finder`. Any application that can control the Finder (listed in `Automation` in the `Privacy` pane) also has `Full Disk Access`, although you will see neither the Finder nor the controlling app listed in the `Full Disk Access`.

**In conclusion**, Transparency, Consent, and Control technology is a vital component of the macOS privacy framework, providing users with the ability to control access to their personal information by apps, ensuring transparency and consent in the process. However, users should also exercise caution when granting full disk access to any app, as this can potentially compromise the integrity of TCC databases and the privacy of their personal information.

Overall, TCC technology is an important feature of macOS, but it is just one of many security measures that users should take to protect their personal data. By staying vigilant and informed, users can help ensure that their privacy is not compromised, even as technology continues to evolve.
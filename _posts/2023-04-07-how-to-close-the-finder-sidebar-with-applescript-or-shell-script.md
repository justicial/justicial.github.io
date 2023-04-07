---
layout: post
title: "How to Close the Finder Sidebar with AppleScript or Shell Script"
date: 2023-04-07
categories: AppleScript
---
If you're someone who finds the Finder sidebar to be an annoyance, you're not alone. Many people feel the same way and often wonder what can be done to get rid of it. Fortunately, there is a simple solution that can help you out. By using either AppleScript or shell script, you can easily close the sidebar and get back to your work without distractions.

Let's start with the **AppleScript** code. This script will tell the Finder to close the sidebar of every window that is currently open:

{% highlight AppleScript %}
tell application "Finder"
	repeat with fWindow in (get every Finder window)
		set sidebar width of fWindow to 0
	end repeat
end tell
{% endhighlight %}

This script will loop through every open Finder window and set the sidebar width to zero, effectively hiding it from view. If you're not familiar with AppleScript, you can simply copy and paste this code into the Script Editor app (found in the Utilities folder) and then run it by pressing the "Play" button.

Alternatively, you can use a one-liner **shell script** that uses the `osascript` binary to run the same AppleScript code:

{% highlight sh %}
/usr/bin/osascript -e 'tell application "Finder" to repeat with fWindow in (get every Finder window)' -e 'set sidebar width of fWindow to 0' -e 'end repeat'
{% endhighlight %}

This script accomplishes the same thing as the AppleScript code, but in a slightly different way. You can save this script as a file with a `.sh` extension and then run it from the command line or add it to any automation shell script that runs on login.

Overall, whether you prefer using AppleScript or shell script, there are easy ways to close the Finder sidebar and get back to your work without distractions. With these simple scripts, you can streamline your workflow and make the most of your time on your Mac.
---
layout: post
title: How to know what user accounts exist on a Mac
date: 2023-04-03 14:00:00
categories: shell-scripting
---

If you're a Mac user, you might be wondering how to find out what user accounts exist on your machine. There are a couple of different ways to do this, and in this post, we'll explore three of them.

## Using who

One way to see what user accounts are currently logged in to your Mac is to use the `who` utility. Running `/usr/bin/who` in the Terminal will display a list of all users currently logged on, showing for each user the login name, tty name, the date and time of login, and hostname if not local.

To see the most information `who` can provide, you can run `who -a`. Filtering the output, you can get a list of usernames by running:
{% highlight sh %}
who | grep "console" | cut -d " " -f 1
{% endhighlight %}

## Using dscl

Another way to see a list of user accounts is to use the `dscl` utility, which stands for Directory Service command line utility. `dscl` is a general-purpose utility for operating on Directory Service directory nodes. Its commands allow one to create, read, and manage Directory Service data.

To see a list of all unique users on your Mac, you can run:
{% highlight sh %}
dscl . list /Users UniqueID
{% endhighlight %}

However, many of the UIDs listed are used by the system rather than console users. To filter out system-related accounts, you can run:
{% highlight sh %}
dscl . list /Users UniqueID | grep -v ^_
{% endhighlight %}

It's important to note that using an underscore as a reliable filter isn't the right decision, as console users can also have an underscore at the start. Instead, you can use the UID as a filter since the UID of interactive users starts with 500. You can use the following command to filter by UID:
{% highlight sh %}
dscl . list /Users UniqueID | awk '$2 >= 500 { print $1 }'
{% endhighlight %}

## Using dscacheutil

Finally, you can use the `dscacheutil` utility to gather information, statistics, and initiate queries to the Directory Service cache. To get a list of all users on your Mac using `dscacheutil`, run:
{% highlight sh %}
dscacheutil -q user
{% endhighlight %}


This will display a list of all users in a specific format. To filter out system-related accounts and show only user accounts with a UID of 500 and higher, you can run:
{% highlight sh %}
dscacheutil -q user | grep -A 4 -B 2 -e uid:\ 5'[0-9][0-9]'
{% endhighlight %}

Knowing what user accounts exist on your Mac can be useful for various reasons, such as troubleshooting, security, and system administration. Try out these methods and see which one works best for you.
---
layout: post
title: "Using trap to Execute Actions on Shell Signal Reception"
date: 2023-04-13 13:00:00 +0300
categories: shell-scripting
---
**trap** is a utility that enables the execution of a specified action when the shell receives a signal. It can be particularly useful for cleaning up resources when a script exits. In this post, we will delve into the syntax of `trap`,  discuss its use cases, provide examples to help you better understand its functionality, and specifically address signals in macOS.

## Signals in macOS

Signals in macOS are similar to those in other Unix-based operating systems. They are used for inter-process communication and allow processes to react to specific events. Some common signals include:

* `SIGHUP` (1): Hangup detected on controlling terminal or death of controlling process
* `SIGINT` (2): Interrupt from keyboard (Ctrl+C)
* `SIGQUIT` (3): Quit from keyboard (Ctrl+\\)
* `SIGABRT` (6): Abort signal from `abort(3)`
* `SIGKILL` (9): Kill signal
* `SIGTERM` (15): Termination signal

For a complete list of signals in macOS, refer to the [signal(3) man page](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man3/signal.3.html).

## Syntax

{% highlight sh %}
trap command signal(s)
{% endhighlight %}
* `command`: The action to be executed when the shell receives a signal.
* `signal(s)`: A list of signals that can be specified by name or by number. Names are case insensitive and should not include the SIG prefix.

> Note: SIGKILL and SIGSTOP signals cannot be intercepted.

## Specifying Command as a String

{% highlight sh %}
trap 'echo "Signal received"' INT
{% endhighlight %}

## Specifying Command as a Function

{% highlight sh %}
cleanup() {
    echo "Signal received"
}
trap cleanup EXIT INT
{% endhighlight %}

## Exiting the Script after Cleaning up Resources

{% highlight sh %}
cleanup() {
    echo "Signal received"
    exit
}
trap cleanup EXIT
{% endhighlight %}

## Ignoring Signals
{% highlight sh %}
trap '' INT
{% endhighlight %}

## Handling Signals in Subshells

{% highlight sh %}
(
    echo "parent process: $$"
    trap 'echo "Signal received"' EXIT
)
{% endhighlight %}

## Sending Signals with kill Command

{% highlight sh %}
kill -KILL $$
{% endhighlight %}

## Testing trap with a script

Create a script trap.sh:
{% highlight sh %}
#!/bin/sh

cleanup() {
    echo "Signal received"
    exit
}

trap cleanup EXIT INT

main() {
    echo "pid: $$"
    while true; do
        echo "sleeping"
        sleep 5
    done
}
main
{% endhighlight %}

Run the script from Terminal:
{% highlight sh %}
sh ./trap.sh
{% endhighlight %}


 Or using launchd:
{% highlight sh %}
launchctl submit -l com.trap -p ./trap.sh -o ./out.log -e ./error.log
{% endhighlight %}

Find the trap.sh process ID (pid):
{% highlight sh %}
ps aux | grep trap | grep -v grep
{% endhighlight %}

Send a signal to the process using the pid:
{% highlight sh %}
kill -INT <pid>
{% endhighlight %}

**In summary**, the `trap` utility provides a way to handle signals in shell scripts, allowing for resource cleanup and other operations to be performed when specific signals are received. By using the examples provided in this post, you can effectively utilize `trap` in your own scripts.
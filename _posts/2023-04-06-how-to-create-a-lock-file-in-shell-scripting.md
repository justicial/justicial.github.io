---
layout: post
title: "How to Create a Lock File in Shell Scripting"
date: 2023-04-06
categories: shell-scripting
---
Preventing a script from running multiple times is a common task in the world of scripting. This is often necessary to avoid race conditions and other undesirable effects that can occur when multiple instances of a script run simultaneously. One way to accomplish this is to use a lock file.

A lock file is simply a file that is used to indicate whether or not a script is already running. If the lock file exists, the script knows that another instance is already running and it can exit gracefully. If the lock file does not exist, the script creates it and begins running.

In this post, we will demonstrate how to **create a lock file in shell scripting** using shell primitives. Here is an example script that demonstrates this functionality:
{% highlight sh %}
#!/bin/sh

LOCK_NAME="$(basename "${0}")" || exit
readonly LOCK_NAME
readonly LOCK_PATH="${TMPDIR}${LOCK_NAME%.*}.lock"

## Lock

add_lock() {
    # check if lock already exists
    ! [ -f "${LOCK_PATH}" ] || return

    # create lock file with process id inside it
    echo "$$" > "${LOCK_PATH}"
}

remove_lock() {
    # check if lock contains process id of the current process to prevent removal by another process
    [ "$(cat "${LOCK_PATH}")" = "$$" ] || return

    rm -rf "${LOCK_PATH}"
}

cleanup() {
    remove_lock
}

# register the cleanup function to be called on the EXIT signal
trap cleanup EXIT

main() {
    add_lock || return

    # perform the script logic
    sleep 70
}
main "${@}"
{% endhighlight %}

Let's take a closer look at how this script works.

First, we define the name and path of the lock file. The name is based on the name of the script, and the path is the `TMPDIR` directory plus the lock file name.

Next, we define two functions: `add_lock` and `remove_lock`. The `add_lock` function checks if the lock file already exists. If it does, the function returns immediately, indicating that another instance of the script is already running. If the lock file does not exist, the function creates it and writes the process id of the current script instance to the file.

The `remove_lock` function checks if the lock file contains the process id of the current script instance. If it does not, the function returns immediately, indicating that another process has created the lock file. If the lock file does contain the process id of the current script instance, the function removes the lock file.

We also define a `cleanup` function that calls `remove_lock` when the script exits. This ensures that the lock file is always removed, even if the script exits unexpectedly.

Finally, we define the `main` function that calls `add_lock` and then performs the script logic. In this example, the script simply sleeps for 70 seconds, but in a real script, this would be replaced with the actual logic of the script.

**In conclusion**, implementing a lock file in your scripts is a simple and effective way to prevent race conditions and other issues that can occur when multiple instances run simultaneously. By using the shell primitives we've outlined in this post, you can easily create a lock file in your scripts and enjoy the peace of mind that comes from knowing your scripts are running as intended.

We hope this post has been helpful and informative. If you have any questions or feedback, feel free to leave a comment below. Thanks for reading!
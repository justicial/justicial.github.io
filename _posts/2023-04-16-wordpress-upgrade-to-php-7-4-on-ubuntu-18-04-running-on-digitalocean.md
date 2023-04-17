---
layout: post
title: "WordPress: Upgrade to PHP 7.4 on Ubuntu 18.04 Running on DigitalOcean"
date: 2023-04-16 18:00:00 +0300
categories: WordPress
---
## Introduction

In this tutorial, we will guide you on how to upgrade to PHP 7.4 on an Ubuntu 18.04 server running on DigitalOcean for your WordPress website. Upgrading to PHP 7.4 will enhance your website's performance, security, and compatibility with new plugins and themes.

The steps provided in this tutorial are based on [Daniel Brinneman's guide](https://danielbrinneman.com/update-to-php-7-4-on-ubuntu-18-04-on-digital-ocean-for-wordpress/). Let's get started!

## Step-by-step Guide

### Step 0: Create a Snapshot (Optional)

Before starting the PHP upgrade process, it's a good practice to create a snapshot of your Digital Ocean droplet. A snapshot is a point-in-time copy of your droplet that you can use to restore your droplet to its previous state if something goes wrong during the upgrade process.

Digital Ocean recommends turning off your droplet before taking a snapshot to ensure data consistency. To create a snapshot, follow these steps:

1. Log in to your Digital Ocean account.
2. Navigate to the `Droplets` section.
3. Select the droplet you want to create a snapshot for.
4. Power off the droplet by clicking on the `On/Off` toggle button.
5. Wait for the droplet to shut down completely.
6. Click on the `Snapshots` tab.
7. Provide a name for your snapshot and click on the `Take live snapshot` button.
8. Wait for the snapshot creation process to complete.
9. Power your droplet back on by clicking on the `On/Off` toggle button again.

Once you have created a snapshot and your droplet is back online, proceed with the PHP upgrade steps below.

### Step 1: Update Your System

First, ensure your system is up-to-date by running the following commands:
{% highlight sh %}
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo apt autoremove
sudo reboot
{% endhighlight %}

### Step 2: List All Installed PHP Packages

List all installed PHP packages and temporarily save the list to a file:
{% highlight sh %}
sudo dpkg -l | grep php | tee packages-first.txt
{% endhighlight %}

### Step 3: Add PHP 7.4 Repository

{% highlight sh %}
sudo add-apt-repository ppa:ondrej/php
{% endhighlight %}

### Step 4: Install PHP 7.4 and Its Required Extensions

{% highlight sh %}
sudo apt install php7.4 php7.4-common php7.4-cli
sudo apt install php7.4-bcmath php7.4-bz2 php7.4-curl php7.4-intl php7.4-mbstring php7.4-mysql php7.4-readline php7.4-xml php7.4-zip
{% endhighlight %}

### Step 5: List All Installed PHP Packages Again

{% highlight sh %}
sudo dpkg -l | grep php | tee packages-second.txt
{% endhighlight %}

### Step 6: Install PHP 7.4 FPM and Enable Its Configuration

{% highlight sh %}
sudo apt install php7.4-fpm
sudo a2enconf php7.4-fpm
{% endhighlight %}

### Step 7: Verify That PHP 7.4 Is Installed

{% highlight sh %}
php -v
{% endhighlight %}

### Step 8: Disable PHP 7.2 and Enable PHP 7.4

{% highlight sh %}
sudo a2dismod php7.2
sudo a2enmod php7.4
{% endhighlight %}

### Step 9: Restart the Apache Server

Finally, restart the Apache server for the changes to take effect:
{% highlight sh %}
sudo systemctl restart apache2
{% endhighlight %}

## Conclusion

Congratulations! You have successfully upgraded to PHP 7.4 on your Ubuntu 18.04 server running on Digital Ocean for your WordPress website. Make sure to test your website thoroughly to ensure everything works as expected. Enjoy the improved performance, security, and compatibility that PHP 7.4 has to offer!
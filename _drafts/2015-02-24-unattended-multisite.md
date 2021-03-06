---
layout: post
title: "Issues with unattended multisite install"
modified:
categories: drupal
excerpt:
tags: []
image:
  feature:
date: 2015-02-24T17:06:04+00:00
comments: yes
---

Apologies, this is probably the most rarefied and pedantic post I have ever written.

I recently had some issues running an unattended drupal install with 

I found that ``/includes/install.core.inc`` which does all the magic in my php script, was not willing to install the site with my test database (used by the second multisite **dev2**). After some fiddling I found that the [install_drupal](http://bit.ly/1Bmelfy) function does not use the db settings from the $settings variable I passed it (though required to validate the form), and instead looked for relevant settings.php file (using [conf_path](http://bit.ly/1BmjjZT) to connect to the db. If I ``require_once("mysettings.php")`` the unattended install still will work butu only if I delete or rename the ``sites/default/settings.php`` file.

I have tried a couple of things to make the above work in a drupally fashion, such as attempting to manually set ``$base_path``. I decided not to get bogged down with this small detail however, and in the end settled for fixing the issue by using a hacky little bash script to hide the default settings before running the install. The bash script is housed in the ``sites/<multisite-domain>`` folder.

{% highlight bash %}

#!/bin/bash
# set current directory 
DIR=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )
echo "running install script from "$DIR
# wipe the slate clean
mysql -u resourceful_temp -e "DROP DATABASE resourceful_temp; CREATE DATABASE resourceful_temp;"
echo "db wiped"
# hide the default settings.php
# so install profile can use our settings.php instead
sudo mv $DIR/../default/settings.php $DIR/../default/settings.hidden.php && echo "moved settings"
php $DIR/temp.install.php && echo "attempted db install"
# restore the default settings.php
sudo mv $DIR/../default/settings.hidden.php $DIR/../default/settings.php && echo "restored settings"

{% endhighlight %}

The drupal.org [multisite page](http://bit.ly/1BmoKI8) (end of article) mentions that drush doesn't have much support for multisites. I will take this to suggest that other automated processes, like my install script, might also have issues inter-operating with multisites, hopefully it won't cause me any more issues during development.


---
layout: post
title: "Making a Drupal Distribution #1"
modified: 2015-24-23T16:26:33+00:00
categories: drupal
excerpt: "<p>Recreating and simplifying a multiple site platform by creating a re-usable distribution.</p> <p>#1 Getting started.</p>"
tags: [drupalplanet]
date: 2015-02-23T16:59:33+00:00
comments: yes
---

##Starting a Drupal Distribution / Install Profile

I have made drupal install profiles for projects before, but that was some while ago. The process relied quite heavily on features, and I found that difficulties [easily] capturing configuration in this way, paired with a lack of time, led my nice clean distribution to diverge over time from the live site it was intended to [re]create. So distros fell by the wayside.

I have reached a point where I feel I must clean up a [multiple site setup](http://tombola.github.io/drupal/drupal-distribution/), and in the process I would like to end up with a better documented and fully reproducable final product, aimed at adaption and re-use.

I am effectively going to rebuild the sites into a single, simpler site (using Spaces), and exile extraneous functions to loosely coupled apps, where they will hopefully have less of an impact on the maintainability of the site. I will be building from all that I have learnt about the platform's usage and re-using components, so fingers crossed this will not be nearly as painful as the slow requirements-led iterative creation of the individual sites first time round.

I want to wrap this all up neatly (and under version control), so I am re-visiting install profiles, features and distributions in order to get the configuration into [re-usable] code.


##Capturing the functionality

My new distribution based site will aim to reproduce all the functionality of the existing sites. The substantial underlying differences in [the platform](http://tombola.github.io/drupal/drupal-distribution/) should be invisible to site visitors, and besides some ui improvements, transparent to site administrators.

I will start by capturing the functionality of the MVP version of my site.

For the first step, I created a make file from the development clone of the production site to use in the installation profile.

{% highlight bash %}
    $ drush generate-makefile base.make --exclude-versions
{% endhighlight %}

This will get all the relevant modules installed, which is a good start, though obviously I will then need to get them all configured to replicate the functionality of my site. That is where [Features](drupal.org/project/features) will come in.

At this point I will admit, my site platform makes use of too many contrib modules, added over time to solve various workflow issues. For the purposes of my own sanity, I am going to start off by commenting out all but the most integral modules in the make file, and then grouping them so that I can tackle rebuilding configuration in a more organised fashion:

- Core functionality (eg views, ctools)
- Administration (eg admin menu, vbo)
- Development (eg devel, features)
- Presentation (eg ds, context)
- Integration (eg feeds)
- Search
- Reporting

When I come out the other end of the rebuild I hope to have made all the difficult decisions and dropped most of the modules that crept in. My reborn Phoenix of a site should therefore be able to fly (read 'Performative', I couldn't resist the allegory).


## Development Environment

To work through the process of building up my existing site (with modifications) into a usable distribution, I need to make some provision within my local development environment. Alongside my regular local development install of the site, I will need a fresh new install (dev1) to start building up the functionality, and a disposable install (dev2) that I can repeatedly run the installation profile on to troubleshoot and run functional tests. The process will be:

1. build functionality into **dev1** (to match live)
1. capture config using make file and features
1. <del>git commit **dev1**</del>\*
1. <del>git update **dev2**</del>\*
1. run the dev2 install profile
1. check config/functionality preserved accurately
1. record functional test (probably selenium)

<a name="multisite"></a>
<div class="post__update"></div>

\* I am writing this post whilst figuring out the steps I describe (it is quite helpful to me). It has just occurred to me that dev1 and dev2 could, and probably should, share the same code base, rather than relying on me to keep them in sync. I think this would require me to use *multisites*. To this end I will point two local domains at the site and use drupal's multisite to have a separate db for **dev2**.

<a name="unattended"></a>
I will want to be able to rattle through the installation without having to fill in forms, so I will be running a [non-interactive installation](http://heyrocker.com/content/installing-drupal-7-non-interactively) from the command line, utlising ``install_drupal``. this method will be facilitated by a php script in the root of my installation, which has the significant advantage that it will also be under version control. I could run the install directly from the [commandline](http://www.coderintherye.com/install-drupal-7-using-drush) using drush [site-install](http://www.drushcommands.com/drush-6x/core/site-install), but then I would have to configure my install with ``key=value`` rather than a nice neat PHP array, which could get complicated once I include my own distribution option pages.

``php install.test.php``


{% highlight php %}

<?php

  define('MAINTENANCE_MODE', 'install');
  define('DRUPAL_ROOT', getcwd());

  // load the db settings for test site
  require_once DRUPAL_ROOT . '/sites/test/settings.php';
  require_once DRUPAL_ROOT . '/includes/install.core.inc';

  $settings = array(
  'parameters' => array(
    'profile' => 'minimal', // to be replaced with my own profile
    'locale' => 'en',
  ),
  'forms' => array(
    'install_settings_form' => array(
      'driver' => 'mysql',
      'database' => 'my_db',
      'username' => 'my_db_user',
      'password' => '',
    ),
    'install_configure_form' => array(
      'site_name' => 'resourceful test',
      'site_mail' => 'info@site.com',
      'account' => array(
        'name' => 'root',
        'mail' => 'info@site.com',
        'pass' => array(
          'pass1' => '1234',
          'pass2' => '1234',
        ),
      ),
      'update_status_module' => array(
          1 => TRUE,
          2 => TRUE,
        ),
      'clean_url' => TRUE,
    ),
  ),
);

install_drupal($settings);

?>
{% endhighlight %}

To run the install profile again I will need to drop all the tables in the database (by recreating it):

{% highlight mysql %}
  mysql -u my_db -e "DROP DATABASE my_db; CREATE DATABASE my_db;"
{% endhighlight %}

----

Anyway, this forms my starting point, next up will be creating and capturing some config (with features), and adding those details to my make file in an install profile.

I will write up any further steps (and inevitable issues) as I get to them (and get a minute).

<div class="btn btn-info btn-disabled">Next exciting installment</div>

---
title: "Getting started with Drupal"
date: "2018-01-26"
layout: post
author: Shannu
category: 
    - Drupal
    - OSS
thumbnail: /assets/img/posts/drupal-logo.png
---


**About Drupal:**

Drupal is a web platform which works on Content Management System (CMS), which helps to create and manage digital content.This system actually helps one who cannot understand code or doesn't know how databases work to manipulate the database, adding templates, designing the layout with the user interface.Here are some examples of what can we create using Drupal: [Barack Obama](https://barackobama.com/), [themis](http://themis.asu.edu/).

**Getting started with Drupal:**

To contribute or to fix any bugs in any organization we should first,  have the source code of the project we wanted to work in our local system.Now we are going to learn how to install Drupal on our localhost using Linux machine(the os I used is Ubuntu 16.04).Before starting log into the following sites  [drupal.org](http://drupalladder.org) and [Drupal ladder](http://drupalladder.org/ladder/ee503327-50be-1904-8d04-9499098cad64).

### STEP 1:

**Install and configure Drupal local environment using LAMP stack in Ubuntu.**

Before installing let us know what LAMP STACK is? and what does it do?

**LAMP STACK:** LAMP stands for (Linux, Apache, MySQL, PHP) which is a platform useful manage websites.

![LAMP_software_bundle.png](/assets/img/posts/lamp.png){:class="img-fluid"}

The above picture lets you understand the workflow of the LAMP to some extent.Here **Linux** is our server side, **Apache** is the web server which deals with the HTTP requests and the content written in php. **MySQL** RDMS (Relational Database Management System) which is based on SQL and as Database management system.**PHP** as the Object-oriented scripting language.

**The installing LAMP in Linux:**

There are many ways to install LAMP

**Install** **APACHE2:**
{% highlight bash %}
$ sudo apt-get update
$ sudo apt-get install apache2
{% endhighlight %}

after this, you need to install MySQL

**_Install MySQL:_**
{% highlight bash %}
$ sudo apt-get install mysql-server mysql-client
{% endhighlight %}
**Installing PHP:**
{% highlight bash %}
$ sudo apt-get install mysql-server mysql-client
{% endhighlight %}
OR there is a single line command with which you can install the LAMP in your Linux pc and install **phpmyadmin**.
{% highlight bash %}
$ sudo apt-get install lamp-server^ phpmyadmin
{% endhighlight %}
PHPMyAdmin is again a free tool which allows us to manage databases (create and modify) and it can be used to administrate MySQL over the Web.

#### **STEP 2:**

**Start Apache and MySQL servers.**

You can do it using the "start" command.
{% highlight bash %}
$ sudo service apache2 start
$ sudo service mysql start
{% endhighlight %}
**STEP 3:**

**Clone Drupal8 to 'drupal8' folder in /var/www/html**
{% highlight bash %}
$ cd /var/www/html
{% endhighlight %}
use git to clone Drupal8.If you don't have git you can install it by using
{% highlight bash %}
$ sudo apt-get install git
{% endhighlight %}
You should then configure git.

When you are sure git is installed follow the command line below:

{% highlight bash %}
$ git clone --branch 8.5.x http://git.drupal.org/project/drupal.git drupal8
{% endhighlight %}

Next, you need to give Read and Write permissions to 'sites/default' and all its subfolders which can be done by using chmod command.

There is one more step to follow to proceed, that is installing the composer.

**Composer:**

The composer is a PHP dependency manager which will manage the dependencies on a project by the project basis.it will install required dependencies to make the project work and it does the installation on its own.You can use the below command to install composer.
{% highlight bash %}
$sudo apt-get install curl php5-cli git
{% endhighlight %}
we are installing php5-cli to install composer and run it.When php5-client is installed you can move on to installing composer using curl.
{% highlight bash %}
$ curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer 
{% endhighlight %}
then test the installation by running  $ composer in your terminal and you will see the following output.

![Screenshot from 2018-01-25 21-14-45](/assets/img/posts/screenshot-from-2018-01-25-21-14-45-e1516895195271.png){:class="img-fluid"}

with all things set we can move on to the next step of building Drupal.

**STEP 4:**

**Building Drupal to localhost.**

Now move to drupal8 directory and install all the dependencies with the composer which can be done by the following commands
{% highlight bash %}
$ cd drupal8
$ composer update
{% endhighlight %}
When all dependencies got installed you should now copy 'default.settings.php' to 'settings.php' in the same directory.This file can be found in the default folder which we gave Read and Write permissions.(you can use 'chmod' command for it)

Now we can create a new database using PHPmyadmin.Open that in your favorite browser by

`localhost/phpmyadmin`

By using the above link in the browser you will get a login page and once logged in with the credentials you gave during installing you can see something like this.

![screenshot-from-2018-01-25-21-37-14.png](/assets/img/posts/screenshot-from-2018-01-25-21-37-14-e1516896696706.png){:class="img-fluid"}

Click on 'Databases' in the top left corner and create a new database with name 'drupal8' and select UTF Unicode in the drop-down menu next to it.

Congratulations! you have created a database.Now to see your database use

`localhost/drupal8`

You will get a page Shown as below fill in the details and you are done.

![screenshot-from-2018-01-26-11-37-08.png](/assets/img/posts/screenshot-from-2018-01-26-11-37-08-e1516947709770.png){:class="img-fluid"}
Congratulations you have setup drupal locally.

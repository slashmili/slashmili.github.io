---
layout: post
title: "Dependency Manager for PHP"
date: 2014-11-23 20:14
comments: true
categories:
  - php
  - composer
---

##What is dependency manager?

Let's say in your application you need to send a tweet from your Twitter account, what would you do? There are a many [PHP Twitter client](https://dev.twitter.com/overview/api/twitter-libraries#php) out there.

The quick way is just download one of them and put all the files in your application root. Well not a smart choice! You are adding someone code to your code base you need to maintain it, update it and so forth. What if there is a way that you say I want Library XYZ in my project version 2.0 and everything would be ready magically!

Say welcome to [Composer](https://getcomposer.org/).



<!-- more -->

##What is composer?

Composer is a tool that helps you to manage your PHP project dependency and setup and install PHP library that other developers shared in https://packagist.org/


##How to use it

{% codeblock %}
$ curl -sS https://getcomposer.org/installer | php
{% endcodeblock %}

And let's say I pick [codebird-php](https://packagist.org/packages/jublonet/codebird-php) to talk to Twitter API. I'd go to the [library's page in packagist](https://packagist.org/packages/jublonet/codebird-php) and check the available version. Right now version _2.6.0_ is the latest. I create a file named _composer.json_ in my root of project:

{% codeblock %}
{
    "require": {
        "jublonet/codebird-php": "2.6.0"
    }
}
{% endcodeblock %}



{% codeblock %}
$ php composer.phar install
Loading composer repositories with package information
Installing dependencies (including require-dev)
  - Installing composer/installers (v1.0.18)
    Downloading: 100%

  - Installing jublonet/codebird-php (2.6.0)
    Downloading: 100%

Writing lock file
Generating autoload files
{% endcodeblock %}


And that's it! If you look at the directory that composer.json is saved, you'll find out that a new file *composer.lock* and a new directory *vendor* are created after you ran the above command.

To use the new library you need to include _vendor/autoload.php_ to your PHP files:

{% codeblock %}
//app.php
require_once __DIR__ . 'vendor/autoload.php'
$cb = Codebird\Codebird::getInstance();

...
{% endcodeblock %}

As you can see you don't need to worry about place that the library is downloaded all you need to do is just include _vendor/autoload.php_ and use the libraries in your application.


##Should I use it?

Definitely! Tools for managing dependencies are de facto in writing and building modern applications.

Other languages Like Perl, Python, Ruby, Java, JavaScript they all have their own tools to manage dependencies for their program and it's proven that improves productivity and helps to build reusable software.


##What's next?

Composer is a powerful tool and it's easy to mix up the idea and miss use. In the next post I'll talk about the best practices in using composer.

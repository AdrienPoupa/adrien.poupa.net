---
title: 'Modern PHP Development for WordPress'
date: '2020-02-01T14:00:00-05:00'
author: Adrien Poupa
url: /modern-php-development-for-wordpress/
categories:
    - PHP
---

Let’s face it, WordPress’ reputation among PHP developers is atrocious. It is, for the most part, justified by the questionable code quality of most plugins (because the barrier entry is very low), the will of the core developers to keep [backwards compatibility](https://mariopeshev.com/how-does-wordpress-maintain-backward-compatibility-over-time/) at all cost, and, as a result, its [architecture](https://nehalist.io/why-im-not-the-biggest-wordpress-fan/#reason1itsarchitecture).

However, as you may have noticed, this blog is running WordPress. Why would you ask me? Well, despite all its flaws, WordPress is stupidly easy to use and complete. I considered using Jekyll on GitHub pages but realized that I would miss many features that I like. I can have comments without relying on external services like Disqus, after all I start to appreciate Gutenberg, I do not have to rely on third-party providers for something as simple as a [contact form](https://jekyllthemes.io/resources/jekyll-contact-forms/)… and I’m OK with running a server for my blog as it would run anyway.

That being said, there are a couple of things you can do to make your developer experience pleasant, or at least way less painful than you would think maintaining a WordPress website.

## Keep it simple, stupid

First, try to install as few plugins as possible. This is because most of them are badly done or really overkill for your needs. For example, contact forms. The reference in the WordPress ecosystem is [Contact Form 7](https://wordpress.org/plugins/contact-form-7/). But if you only need the simplest contact form, do you really need a full blown plugin that will allow you to customize every detail of your form? This is why I decided to write my own contact form plugin, with just enough code to display a form and process it. I used [this](https://www.sitepoint.com/build-your-own-wordpress-contact-form-plugin-in-5-minutes/) as base and it served me well.

I wanted to use SMTP to send emails. Rather than using the huge [WP Mail SMTP](https://wordpress.org/plugins/wp-mail-smtp/) plugin, since WordPress already includes PHPmailer, why not write another one-file plugin?

```
<?php
/**
 * Plugin Name: PHPMailer SMTP
 * Plugin URI: https://adrien.poupa.net
 * Description: Override WordPress PHPMailer settings.
 * Version: 1.0
 * Author: Adrien Poupa
 * Author URI: https://adrien.poupa.net
 */
add_action( 'phpmailer_init', 'send_smtp_email' );
function send_smtp_email( $phpmailer ) {
	$phpmailer->isSMTP();
	$phpmailer->Host       = SMTP_HOST;
	$phpmailer->SMTPAuth   = SMTP_AUTH;
	$phpmailer->Port       = SMTP_PORT;
	$phpmailer->Username   = SMTP_USER;
	$phpmailer->Password   = SMTP_PASS;
	$phpmailer->SMTPSecure = SMTP_SECURE;
	$phpmailer->From       = SMTP_FROM;
	$phpmailer->FromName   = SMTP_NAME;
}
```

Simply define the new constants in your wp-config.php and you’re done.

This reasoning also applies with themes. The more features a theme has, the more complex it is and the more bugs you will probably find. Thus, I’d rather install a very simple one that I can tweak if need be, or write my own from scratch if I have many requirements.

In general, if what you want to do is really simple, maybe you should take the 30 minutes to write your plugin rather than relying on a 5-year old plugin filled with bugs.

## Version your Code

You should version your WordPress installation like any other project, but no need to keep track of the core files or the plugins. [This gitignore](https://gist.github.com/redoPop/444295) is great: it only versions what matters, ie the plugins or themes that you define.

Because you do not want to version your static content, I would advise to store your media in a CDN. S3 is great for this purpose. I use the [S3-uploads](https://github.com/humanmade/S3-Uploads) plugin and so far it served me well.

## Use the Command Line to Manage Your Dependencies

Like on every other project I do, I want to be able to manage my dependencies through Composer. This way I can keep track of what is installed, when it was installed, what is the current version, etc. Thankfully, the excellent [WordPress Packagist](https://wpackagist.org/) fills this need. After setting the repository URL in composer.json, installing a plugin or a theme is as easy as

```
composer require wpackagist-plugin/akismet

composer require wpackagist-theme/hueman
```

WordPress itself can be managed via the command line. [WP-CLI](https://wp-cli.org/) comes with many useful [commands](https://developer.wordpress.org/cli/commands/), my favorite being the core update.

```
wp core update
```

## Use a WordPress Framework

If you have a more complex project to setup and you have to do it with WordPress, or you are more comfortable with WordPress, these tweaks will only take you so far.

Although I never used it, the [Bedrock framework](https://roots.io/bedrock/) seems to bring a better structure to WordPress and bring it to modern age: Composer-enabled, .env file for credentials, better directory structure, CLI tools… I would use it to do any WordPress project more complex than a blog. To create clean themes, [Sage](https://roots.io/sage/) is also a great option.
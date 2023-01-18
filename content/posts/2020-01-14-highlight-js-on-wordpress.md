---
title: 'Using highlight.js on WordPress'
date: '2020-01-14T19:27:41-05:00'
author: Adrien Poupa
url: /highlight-js-on-wordpress/
categories:
    - PHP
---

I was looking for a replacement for the syntax highlighter I was using on my blog, since [SyntaxHighlighter Evolved](https://wordpress.org/plugins/syntaxhighlighter/) never really gave me satisfaction (unwanted advertisements next to the code blocks, poor styling).

I wanted a lightweight solution and found [highlight.js](https://highlightjs.org/). It highlights blocks on code on the client side and can be enabled with [just a few lines](https://highlightjs.org/usage/):

```
<link rel="stylesheet" href="/path/to/styles/default.css">
<script src="/path/to/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

There used to be a [plugin](https://wordpress.org/plugins/wp-code-highlightjs) to integrate it to WordPress but it was removed for security reasons (I am not sure what this means).

It does not really matter; since the installation of the library is so easy, I figured I’d write my own plugin. It is as dumb as you would expect since it is only adding the three lines shown above. It loads the latest version of the library and the CSS file from jsdelivr.

To create a code block from Gutenberg, simply start a new paragraph with three back ticks: “` and paste your code within it.

The plugin repository is [here](https://github.com/AdrienPoupa/HighlightJS-WP), but the plugin is so simple that I can paste the code here:

```
<?php
/**
 * Plugin Name: HighlightJS WP
 * Plugin URI: https://github.com/AdrienPoupa/HighlightJS-WP
 * Description: HighlightJS WordPress Integration
 * Version: 1.0.0
 * Author: Adrien Poupa
 * Author URI: https://adrien.poupa.net
 * License: GPL-2.0+
 * License URI: http://www.gnu.org/licenses/gpl-2.0.txt
 */

 // Prevent direct access
if ( ! defined( 'WPINC' ) ) {
	die;
}

add_action( 'wp_enqueue_scripts', 'hjswp_enqueue_wphighlightjs' );

/**
 * Load HighlightJS
 */
function hjswp_enqueue_wphighlightjs() {
	// Enable the plugin only for singular posts
	if ( ! is_singular() ) {
		return;
	}

	wp_enqueue_style( 'highlightjs-css', '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@latest/build/styles/default.min.css' );

	wp_enqueue_script( 'highlightjs', '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@latest/build/highlight.min.js', '', 'latest', true );

	wp_add_inline_script( 'highlightjs', 'hljs.initHighlightingOnLoad();' );
}

```

You may want to modify it if you wish to load a specific version of highlight.js or if you want to use a different theme.

I tried to release it to WordPress.org but apparently I can’t because they “no longer accept frameworks, boilerplates, and libraries as stand-alone plugins”. Oh well, nevermind, just get it from here 🙂
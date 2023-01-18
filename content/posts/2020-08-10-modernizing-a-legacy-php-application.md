---
title: 'Modernizing a Legacy PHP Application'
date: '2020-08-10T20:25:00-04:00'
author: Adrien Poupa
url: /modernizing-a-legacy-php-application/
categories:
    - PHP
---

*Update Aug, 11: This post was well received on [Reddit](https://www.reddit.com/r/PHP/comments/i7wadq/modernize_a_legacy_php_application/), so I added new anti-patterns to reflect the comments.*

Recently, I had the <del>chance</del> occasion to work on numerous legacy PHP applications. I spotted common anti-patterns that I had to fix. This article is not about rewriting an old PHP application to &lt;insert shiny framework name here&gt;, but about how to make it more maintainable and less of a hassle to work on.

## Anti-pattern #1: Credentials in the Code

This is definitely the most common worst pattern I’ve seen. Many projects simply have sensitive information such as database username and password in the versioned code. This is obviously a bad practice because it makes it impossible to create a local setup as the code is tied to a specific environment, and everybody with an access to the code can see the credentials for what’s usually the production environment.

To fix it, my preferred way to proceed that works with any application is to install [vlucas’ phpdotenv package](https://github.com/vlucas/phpdotenv), that allows to create an environment file and access the variables with the environment super variable.

That means we create two files: a `.env.example` file that will be versioned and serve as a template for the `.env` file that will hold the credentials. The`.env` will not be versioned, so add the .env file to your `.gitignore`. The [official documentation](https://github.com/vlucas/phpdotenv#usage) explains this well.

Your .env.example will list the credentials:

```
DB_HOST=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```

Your .env will have the credentials:

```
DB_HOST=localhost
DB_DATABASE=mydb
DB_USERNAME=root
DB_PASSWORD=root
```

In the common file, load the .env file:

```php
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();
```

Then you can access your credentials using `$_ENV['DB_HOST']` for example.

Note that it is not recommended to use the package as is in production, it is better to either:

- Inject the environment variables at the runtime of your container, if you are using a Docker-based deployment, or in the HTTP server config if possible;
- Cache the environment variables to avoid the overhead that comes with reading the .env file with every request; this is [how Laravel does it](https://github.com/vlucas/phpdotenv/issues/207#issuecomment-260116783).

Files that contained credentials can be purged from the Git history [as shown here](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository).

## Anti-pattern #2: Not Using Composer

I found it was very common back in the days to have a `lib` folder with big libraries such as PHPMailer. This is a big no-no when it comes to versioning, so those dependencies should be managed with [Composer](https://getcomposer.org/). This way, you can very easily see which version of a given package you are running, and update them as needed.

So go ahead and start `compose require` all the things.

## Anti-pattern #3: No Local

Most of those applications I worked on only had one environment: production.

![](https://cdn.poupa.net/uploads/2020/08/r_766674_WRjqh.jpg)But now that you fixed Anti-pattern #1, you should be able to setup a local easily. You still may have some hard coded configuration such as upload paths, but now you can move those over to your .env.

To create new local environments, unsurprisingly I use Docker. I find it especially well-suited for legacy projects as they often use older PHP versions that you do not want to or cannot install.

You can use a service such as [PHPDocker](https://phpdocker.io/generator) or use a light docker-compose.yml file.

## Anti-pattern #4: Not Using a Public Folder

I found that most of those old projects are accessible from their root. In other words, any file stored at the root of the project will be publicly readable. This is especially bad when malicious users (ie, kiddies scripts) will try to access included files directly as you may not have defined an `exit` if the script is accessed directly in all of your included files.

This is obviously incompatible with using a .env file or Composer as it is a [bad idea](https://www.reddit.com/r/PHP/comments/8hnpot/a_vendor_directory_should_always_be_outside_the/) to expose the vendor folder. Yes, there are [tweaks](https://stackoverflow.com/a/50766284/11989865) to make this possible but when possible, go ahead and move all the client-facing PHP files to a public folder, and alter your web server configuration to make this the root of your application.

In my typical application, I will have:

- A `docker` folder for Docker-compose related files (Nginx configuration, PHP Dockerfile, etc)
- An `app` folder where the business logic is stored (services, classes, etc)
- A `public` folder where the client-facing PHP scripts are stored as well as the assets (JS/CSS). This is the root of the application as far as the client is concerned.
- `.env` and `.env.example` files at the root

## Anti-pattern #5: Blatant Security Issues

PHP applications not using a framework, especially old ones, are prone to blatant security issues such as:

- SQL injections made possible by not escaping parameters in query. Use PDO to prevent this!
- XSS injections made possible by not escaping user data when it is displayed. Use `htmlspecialchars` to prevent this.
- File uploads… a topic [of its own](https://www.acunetix.com/websitesecurity/upload-forms-threat/). If the developer implemented his own, custom made solution, it is highly possible that it has one or more security issues.
- [CSRF attacks](https://portswigger.net/web-security/csrf) caused by not validating the origin of a request. I recommend Paragonie’s [Anti-CSRF package](https://github.com/paragonie/anti-csrf) that can be integrated with an existing application easily.
- Poor password encryption. I have seen many projects still using SHA-1 or even MD5 to hash the passwords. PHP offers good BCrypt support out of the box since 5.5, it would be a shame not to use it. To migrate the passwords painlessly, I like to update the hash in the database as the user logs in. You just need to make sure that the password column is long enough to accomodate BCrypt passwords, a VARCHAR(255) is good enough. Some pseudocode to see how it works:

```php
<?php
// Password still in old hash: does not start with $
// Password is correct: convert it and log the user
if (strpos($oldPasswordHash, '$') !== 0 &&
    hash_equals($oldPasswordHash, sha1($clearPasswordInput))) {
    $newPasswordHash = password_hash($clearPasswordInput, PASSWORD_DEFAULT);

    // Update the password column

    // User logged in: return success
}

// Password already converted
if (password_verify($clearPasswordInput, $currentPasswordHash)) {
    // User logged in: return success
}

// User not logged in: return failure
```

## Anti-pattern #6: No Tests

This is very common among legacy apps. While I think it is unrealistic to start writing unit tests for the whole application, one can write functional tests.

![](https://cdn.poupa.net/uploads/2020/08/image-result-for-unit-testing-vs-functional-testin.png)These are high level tests as shown in the image above, that will help us to ensure that subsequent refactoring of the application does not break it. It can be as simple as, for example, firing up a browser and logging to the application, then expecting a HTTP success code and/or a specific success message in the resulting page. PHPUnit can be used to do that, or other tools like [Cypress](https://www.cypress.io/), or [codeception](https://codeception.com/).

## Anti-pattern #7: Poor Error Handling

If (or most likely when) something breaks, you want to be aware of it, quickly. But most legacy apps handle errors poorly, relying on PHP’s forgiveness to carry on no matter what.

You want to be able to catch and log as many errors as possible to correct them. There are [good articles](https://netgen.io/blog/modern-error-handling-in-php) on the subject.

Throwing specific exceptions will also make it easier for you to locate the location of an error.

## Anti-pattern #8: Global Variables

I thought I’d never see those again until I worked on a legacy project. And they make reading and understanding the behavior of code unpredictable. After all, they are [evil](https://softwareengineering.stackexchange.com/a/148109).

It is better to [use Dependency Injection instead](https://softwareengineering.stackexchange.com/a/297935), because it gives you control over which instance is used where. A package like [Pimple](https://github.com/silexphp/Pimple) will do the job well.

## What to Improve Next?

Depending on the future of the application or the available budget, several more steps can be taken to improve it.

First, if the app is running on an ancient version of PHP (anything below 7), you should really consider updating it. Most of the time it should not be really painful, and the longest task will probably be getting rid of all those `mysql_` calls if the application is using them. You can use something like [this library](https://github.com/dshafik/php7-mysql-shim) for a quick and dirty fix, but the best solution would be to rewrite all the queries to PDO, making sure that the parameters are escaped at the same time.

If the app is not using the MVC pattern, ie having the business logic and the templates separated, it might be a good time to introduce a template library (I know, PHP is a templating language but modern libraries are just much nicer) like Smarty, Twig or Blade.

Finally, if you can afford it, a rewrite to a modern PHP framework such as Laravel or Symfony is much better in the long run. You will have all the tools you need to do safe, sane PHP development. Depending on the size of the application, if it is big, I would advise using the [strangler pattern](https://martinfowler.com/bliki/StranglerFigApplication.html) to avoid a [big bang rewrite](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/) that could (and probably would) end badly. This way, you can migrate parts of the code you are currently working on to the new system while keeping the old working ones untouched until you need to work on them.

This is a cost-efficient approach that allows you to have a modern PHP environment for your day to day tasks while avoiding feature freeze for weeks or months depending on the project.

I will be writing another article on how to do this with Laravel soon.

I hope you enjoyed this article and please feel free to comment below!
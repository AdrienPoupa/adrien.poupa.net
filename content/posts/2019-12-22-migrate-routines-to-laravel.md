---
title: 'Migrate Routines to Laravel'
date: '2019-12-22T21:28:38-05:00'
author: Adrien Poupa
url: /migrate-routines-to-laravel/
categories:
    - PHP
tags:
    - Laravel
---

I had to work on a project that made heavy use of MySQL routines: stored procedures, views, functions and triggers. In the process of adding Laravel to the application, I found good packages to [seed the database from existing data](https://github.com/orangehill/iseed), [create models from the existing schema](https://github.com/reliese/laravel) and [create migrations from existing tables](https://github.com/Xethron/migrations-generator).

However, I could not find anything to migrate the existing routines as a migration. I needed this to be able to make the existing application work when seeding a new database.

Because there was a lot of stored procedures and views, and because apparently nobody had to solve this issue before (or at least nobody published their solution :)), I made a script to automatically generate the migrations from the database.

And because I thought that [others](https://github.com/Xethron/migrations-generator/issues/150) may need to do the same thing at some point, I packaged it and made it available for anybody to use it: <https://github.com/AdrienPoupa/migrate-routines>

Installation and usage is as easy as

```
composer require adrienpoupa/migrate-routines --dev
```

```
php artisan migrate:views
```

… and similar commands for procedures, triggers and functions.

Please note that because of how the routines definitions have to be retrieved (in lower level tables such as mysql) you should only used this package in a development environment with a highly privileged user, possibly root on a local machine.

I know this is a very narrow corner case, but feel free to use it! You can open a ticket or a PR if something does not work how it should.
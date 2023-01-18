---
title: 'Zero Downtime Laravel Deployments with Laravel Envoy'
date: '2019-06-02T21:00:15-04:00'
author: Adrien Poupa
url: /zero-downtime-laravel-deployments-with-laravel-envoy/
categories:
    - PHP
tags:
    - Laravel
---

Recently, I was looking for a way to deploy a Laravel application with ease and no headache. After some time, I wrote a very basic Bash script to automate the common deployment tasks: pulling changes from Git, installing Composer dependencies, building NPM scripts, running migrations and so on. But it was still feeling primitive. There had to be a better way.

After some research, I found [Laravel Envoyer](https://envoyer.io/). This service offers everything I needed but I was not ready to pay the costs associated. After further research, I found [Laravel Envoy](https://laravel.com/docs/5.8/envoy). Even though their names are similar, they do not provide the same services: while Envoyer is a full featured product ready to use, Laravel Envoy provides a template for deployment tasks.

The main benefit of using Envoy over a Bash script is its integration with PHP: that makes it possible to use credentials stored in the .env file used by Laravel. It is also possible to specify several environments to deploy the application, and send notifications at the end of the deployments. The documentation is very detailed.

With Envoy, we will be writing tasks that are integrated into stories. For example, the “deploy” story will contain the composer task, NPM task and so on.

On top of that, we want to be able to deploy the application with no downtime. The structure we will have is the following:

- /current
    - symbolic link pointing to /releases/latestdate
- /releases
    - folder containing the releases like 20190602000000, 20190602000001 and so on
- /storage
    - the storage folder used by our application. This one stays between releases, so each releases’ storage folder is actually a symbolic link to this on
- .env
    - the environment file used by our application and linked to by each release like the storage folder

Thus, the server will point to the /current/public folder.

The /current symbolic link we be updated once we have finished to build the application in a separate folder, ie /releases/20190602000000 for example.

 Moving on to the actual Envoy script: we will write a script that:

- fetches the latest code from our Git repository
- install latest composer dependencies
- run NPM install and NPM run
- update the symlinks as explained above
- update the application cache for maximum performance
- run the MySQL migration
- clean the releases folder so we only keep the latest five

The script I will detail below is a compilation of three great scripts I found online at:

- <https://github.com/papertank/envoy-deploy>
- <https://github.com/spatie/blender/blob/master/Envoy.blade.php>
- <https://medium.com/@warrickbayman/zero-down-time-laravel-deployments-using-envoy-b0d1f0fddb81>

First, install Envoy globally as explained in the [documentation](https://laravel.com/docs/5.8/envoy#installation):

```
composer global require laravel/envoy
```

Add composer’s bin directory to your PATH variable:

```
export PATH=~/.composer/vendor/bin:$PATH
```

Add the following entries to our .env file:

```
DEPLOY_USER=SSH user
DEPLOY_SERVER=SSH host
DEPLOY_BASE_DIR=directory where the folder structure explained above will reside
DEPLOY_REPO=Git repository to clone
```

For it to work on your computer, you should use a SSH keyfile to connect to your server and add it to your SSH local configuration so that a “ssh user@host” command connects you in the server. [This](https://linuxize.com/post/using-the-ssh-config-file/) is a great tutorial to achieve this purpose. Long story short:

```
nano ~/.ssh/config
```

```
Host your-host.com
    User your-user
    HostName your-host.com
    IdentityFile /path-to-your-identity-file
```

```
chmod 600 ~/.ssh/config
```

Then, you can create a new file named Envoy.blade.php at the root of your Laravel application.

```
@setup
    require __DIR__.'/vendor/autoload.php';
    $dotenv = Dotenv\Dotenv::create(__DIR__);
    try {
        $dotenv->load();
        $dotenv->required(['DEPLOY_USER', 'DEPLOY_SERVER', 'DEPLOY_BASE_DIR', 'DEPLOY_REPO'])->notEmpty();
    } catch ( Exception $e )  {
        echo $e->getMessage();
    }

    $repo = env('DEPLOY_REPO');

    if (!isset($baseDir)) {
        $baseDir = env('DEPLOY_BASE_DIR');
    }

    if (!isset($branch)) {
        throw new Exception('--branch must be specified');
    }

    $releaseDir = $baseDir . '/releases';
    $currentDir = $baseDir . '/current';
    $release = date('YmdHis');
    $currentReleaseDir = $releaseDir . '/' . $release;

    function logMessage($message) {
        return "echo '\033[32m" .$message. "\033[0m';\n";
    }
@endsetup

@servers(['prod' => env('DEPLOY_USER').'@'.env('DEPLOY_SERVER'), 'localhost' => '127.0.0.1'])

@story('deploy', ['on' => 'prod'])
    git
    composer
    npm_install
    npm_run_prod
    update_symlinks
    cache
    migrate
    clean_old_releases
@endstory

@story('rollback')
    rollback
@endstory

@task('git')
    {{ logMessage("Cloning repository") }}

    git clone {{ $repo }} --branch={{ $branch }} --depth=1 -q {{ $currentReleaseDir }}
@endtask

@task('composer')
    {{ logMessage("Running composer") }}

    cd {{ $currentReleaseDir }}

    composer install --no-interaction --quiet --no-dev --prefer-dist --optimize-autoloader
@endtask

@task('cache')
    {{ logMessage("Building cache") }}

    php {{ $currentReleaseDir }}/artisan route:cache --quiet

    php {{ $currentReleaseDir }}/artisan config:cache --quiet
@endtask

@task('update_symlinks')
    {{ logMessage("Updating symlinks") }}

    # Remove the storage directory and replace with persistent data
    {{ logMessage("Linking storage directory") }}
    rm -rf {{ $currentReleaseDir }}/storage
    ln -nfs {{ $baseDir }}/storage {{ $currentReleaseDir }}/storage

    # Remove the public uploads directory and replace with persistent data
    {{ logMessage("Linking uploads directory") }}
    rm -rf {{ $currentReleaseDir }}/public/uploads
    ln -nfs {{ $baseDir }}/uploads {{ $currentReleaseDir }}/public/uploads

    # Import the environment config
    {{ logMessage("Linking .env file") }}
    ln -nfs {{ $baseDir }}/.env {{ $currentReleaseDir }}/.env

    # Symlink the latest release to the current directory
    {{ logMessage("Linking current release") }}
    ln -nfs {{ $currentReleaseDir }} {{ $currentDir }}
@endtask

@task('migrate')
    {{ logMessage("Running migrations") }}

    php {{ $currentReleaseDir }}/artisan migrate --force
@endtask

@task('npm_install')
    {{ logMessage("NPM install") }}

    cd {{ $currentReleaseDir }}

    npm install --silent --no-progress
@endtask

@task('npm_run_prod')
    {{ logMessage("NPM run prod") }}

    cd {{ $currentReleaseDir }}

    npm run prod --silent --no-progress
@endtask

@task('clean_old_releases')
    # Delete all but the 5 most recent releases
    {{ logMessage("Cleaning old releases") }}
    cd {{ $releaseDir }}
    ls -dt {{ $releaseDir }}/* | tail -n +6 | xargs -d "\n" rm -rf;
@endtask

@task('init')
    if [ ! -d {{ $baseDir }}/current ]; then
        cd {{ $baseDir }}
        git clone {{ $repo }} --branch={{ $branch }} --depth=1 -q {{ $release }}
        {{ logMessage("Repository cloned") }}
        mv {{ $release }}/storage {{ $baseDir }}/storage
        ln -s {{ $baseDir }}/storage {{ $release }}/storage
        ln -s {{ $baseDir }}/storage/public {{ $release }}/public/storage
        {{ logMessage("Storage directory set up") }}
        cp {{ $release }}/.env.example {{ $baseDir }}/.env
        ln -s {{ $baseDir }}/.env {{ $release }}/.env
        {{ logMessage("Environment file set up") }}
        rm -rf {{ $release }}
        {{ logMessage("Deployment path initialised. Run 'envoy run deploy' now.") }}
    else
        {{ logMessage("Deployment path already initialised (current symlink exists)!") }}
    fi
@endtask

@task('rollback')
    cd {{ $releaseDir }}
    ln -nfs {{ $releaseDir }}/$(find . -maxdepth 1 -name "20*" | sort  | tail -n 2 | head -n1) {{ $baseDir }}/current
    echo "Rolled back to $(find . -maxdepth 1 -name "20*" | sort  | tail -n 2 | head -n1)"
@endtask

@finished
    echo "Envoy deployment complete.\r\n";
@endfinished
```

he first time you run the script, you can initialize it with

```
envoy init --branch=YOUR_GIT_BRANCH 
```

Now, you can deploy your application with

```
envoy deploy --branch=YOUR_GIT_BRANCH 
```

You can also specify the directory with –baseDir, otherwise it will default to the one specified in the .env.

If something goes wrong during a deployment, you can rollback to the previous version with:

```
envoy rollback --branch=YOUR_GIT_BRANCH 
```

This will update the /current symlink to the previous release.

Then, you can integrate this with your Git repository to run the deploy command after each commit for example. Happy deployments!
# Setting up a Trellis based site, locally.



**Table of Contents**

* [Setting up a Trellis based site, locally.](#setting-up-a-trellis-based-site-locally)
  * [Prerequisites](#prerequisites)
     * [1.0. Clone the repo](#10-clone-the-repo)
     * [2.0. Add the vault pass file if you need it](#20-add-the-vault-pass-file-if-you-need-it)
     * [3.0. Installing dependencies](#30-installing-dependencies)
        * [3.1. Base dependencies](#31-base-dependencies)
        * [3.2. Must-use plugins](#32-must-use-plugins)
        * [3.3. The theme](#33-the-theme)
     * [4.0. Booting up the VM](#40-booting-up-the-vm)
     * [5.0. Synching the database](#50-synching-the-database)
        * [5.1. Getting wp-admin logins](#51-getting-wp-admin-logins)
        * [5.2. Activating the required plugins](#52-activating-the-required-plugins)
        * [5.3. Syncing the database](#53-syncing-the-database)
     * [6.0. Building the frontend](#60-building-the-frontend)
        * [6.1. Gulp](#61-gulp)
        * [6.2. Webpack](#62-webpack)
           * [6.2.1. NPM](#621-npm)
           * [6.2.2 Yarn](#622-yarn)
     * [Common errors](#common-errors)
        * [503 error when you try to access the site](#503-error-when-you-try-to-access-the-site)

### Prerequisites

Before we start, you'll need a few tools installed and ready to go.

1. [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
2. [Vagrant](https://www.vagrantup.com/intro/index.html)
3. [Composer](https://getcomposer.org/doc/00-intro.md)
4. [Node.js](https://nodejs.org/)
5. [NVM](https://github.com/nvm-sh/nvm)
6. [Yarn](https://yarnpkg.com/en/docs/getting-started)
7. [Gulp](https://gulpjs.com/)
8. [SVGO](https://github.com/svg/svgo#installation)

Make sure that you have these installed prior to starting this guide. It will make your life a **lot** easier if you do!



### 1.0. Clone the repo

Use whatever `git` system you want, but clone the repo locally and open the project root directory in your terminal.



### 2.0. Add the vault pass file if you need it

Some Trellis repos require a file to be added to the `trellis` directory to decrypt passwords.

Trellis is based on Ansible and therefore has access to [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to encrypt files. This allows us to store private information within our git repo, without having to worry about it being read or leaked. Trellis stores this file in the `trellis` directory under a `.gitignore`'d file, generally called `.vault_pass`.

To see if you require a `.vault_pass` file, do the following:

Change into the trellis directory

```bash
cd trellis
```

Check the `ansible.cfg` file for a reference to the `vault_password_file`

```bash
cat ansible.cfg | grep vault_password_file
```

If you don't see anything, then you can skip this step now. 

If you get something like the following returned, then you need to add a file:

```bash
vault_password_file = .vault_pass
```

In this case, we can see that the `vault_password_file` is asking for a file called `.vault_pass`. So we can go ahead and create it now. This can be done by running the following command:

```bash
touch .vault_pass
```

This will have created a `.vault_pass` file and you can now ask your lead developer for the string that will go inside. Simply paste the string your supplied inside this `.vault_pass` file and then forget about it 🙂



### 3.0. Installing dependencies

Whilst trellis will install most dependencies for you, if it fails, then it's a pain to have to manaully install them and re-run the provisioning. For that reason, we'll manually install our dependencies before bringing the box up.



#### 3.1. Base dependencies

`cd` out of the `trellis` directory and into the `site` directory, this is where 99% of our time will be spent.

```bash
cd ../site
```

From here we can run composer and install our PHP dependencies

```bash
composer install
```

This should work off the bat, as long as you have [Composer installed](https://getcomposer.org/doc/00-intro.md).

This command is installing WordPress and any of our selected WordPress plugins. By installing these through composer, we can keep the git repo clean and with a minimum amount of third-party code committed. It also allows us to fix everyone's installs to a certain version of WordPress core and plugins.



#### 3.2. Must-use plugins

The next thing we'll install are the MU plugin dependencies. Generally there will only by one of these in the repo as it's used to encapsulate site functionality (post types, AJAX endpoints etc.). That said, there may be more than one, so don't take that as a given.

To find the plugins you'll need to install dependencies for, `cd` into the `mu-plugins` directory.

```bash
 cd web/app/mu-plugins
```

You can then list all the directories in there

```bash
 ls -dla */
```

Generally, you'll see one named the same as the site you're working on, but either way, `cd` into each directory that's listed and run

```bash
composer install

```

This will install all the dependencies you need for each of the MU plugins.



#### 3.3. The theme

No we'll get into installing our theme dependencies, it's pretty much the same as the MU Plugin.

`cd` back out of the `mu-plugins` directory and into the `themes` directory

```bash
cd ../themes

```

List the sub-directories within the themes directory

```bash
ls -dla */

```

Hopefully, you've only got one directory listed, as a general rule, the theme is named the same as the site you're working on. In this case, we'll use `example-theme` as our theme name.

Move into the theme directory

```bash
cd example-theme

```

Then you can run composer again to install any dependencies.

```bash
composer install

```

Now you should have a fully functional PHP site 🎉



### 4.0. Booting up the VM

Now that we've got all of our dependencies installed, we can go about booting up the VM, using Trellis.

You should already have Virtualbox installed, if you don't make sure you install it **now**!

You should already have Vagrant installed, if you don't make sure you install it **now**!

Move yourself back to the `trellis` directory

```bash
cd ../../../../../trellis 

```



------

**Windows users**

Before running the next command, you'll need to get a couple of other things installed. For some reason, we've found that Trellis doesn't install ansible correctly, so run the following from your WSL (Windows Subsystem for Linux) terminal:

```bash
sudo apt-get install python-pip
pip install ansible

```

------



Now run the following command

```bash
vagrant up

```

This will take some time and you may be asked for your password a couple of times.

The `vagrant up` command will start the process of booting the VM for your local site. It will import a base Linux box and then install all the needed items to get a LEMP stack up and running.

Leave this to complete and you should then be able to access your site locally.

To see what your local URL is, you can run the following command from the `trellis` directory:

```bash
cat ./group_vars/development/wordpress_sites.yml | grep canonical

```

You should be able to see from the output, what your local URL is.



### 5.0. Synching the database

We have the VM up and running, but we have no data. If this is a fresh site with no staging URL, then you can skip this step. If you're coming into the project later and there's a staging site, we can save ourself some trouble and sync all the data down to our local copy.

Log into the WordPress admin panel at the following URL (replace `example.test` with the URL for your local site):

```bash
http://example.test/wp/wp-admin/

```



#### 5.1. Getting wp-admin logins

You can ask your lead dev for the logins for this. Alternatively, you can pull them from the Trellis files:



**Username**

You can get the username by running the follwing command:

```bash
 cat ./group_vars/development/wordpress_sites.yml | grep admin_user

```



**Password**

Getting the password may be more difficult, try this to start with:

```bash
cat ./group_vars/development/vault.yml | grep admin_password

```

If that doesn't work, it's likely that your vault files are encrypted, so we'll need to use `ansible-vault` to read them. Try this:

```bash
ansible-vault view ./group_vars/development/vault.yml | grep admin_password

```

If neither of these work, then you'll need to contact your lead dev to get the login details. 

Hopefully though, you now have a username and password to login to the admin dashboard, go ahead and do this now.



#### 5.2. Activating the required plugins

Now you're in the WordPress admin area, go to **Plugins** in the sidebar.

From here, you'll want to find and activate the following plugins:

- WP Migrate DB Pro
- WP Migrate DB Pro Media Files

Don't worry about the others for now, they'll get activated when we sync the data down 🙂



#### 5.3. Syncing the database

With these plugins active, we can now sync the database from the staging site, to our local copy. Ask your lead developer for the *Migrate DB Connection Info*.

Once you have this, you can go to **Tools > Migrate DB Pro** within the WordPress admin dashboard. 

You'll first need to click on the **Settings** tab and enter a licence key. You should be able to get this from your lead dev.

Once the plugin is activated, head back to the **Migrate** tab and select the **Pull** radio button. A box will open up and ask you for your connection info, paste the string you were given earlier into this box; it should now load some extras.

You can scroll down the page slightly and tick the **Media Files** checkbox, if you'd like to sync media files to your local machine.

I'd also suggest ticking the **Save Migration Profile** checkbox before you start the migration, this will save you from having to do this step again.

You can now happily click the **Save & Migrate** button, to start the migration 🚀



**N.B.** If you run into 503 issues when migrating the images, try running the script again with the **Media Files** checkbox unticked. I've found that sometimes trying to sync the data and media in one fell swoop will cause issues.



### 6.0. Building the frontend

Now your site is up, running and you have data, lets get the frontend looking right.

This could be built in a few ways, so bear with me whilst we work this out!

If the site you're working on is a bit older *(pre mid -2018)* then there's a good chance that it's using the older Gulp.js system. There's nothing wrong with this, other than it's a bit slower than the new Webpack one.

To check if you're using this version, `cd` back into your theme directory. From the root of your project this would be:

```bash
cd site/web/app/themes/example-theme/

```



#### 6.1. Gulp

To check if you're using gulp, run the following command:

```bash
ls | grep gulpfile.js

```

If you get no response, then you're not using gulp, if you see `gulpfile.js` then you are, simple!

With the Gulp system, you simply need to install packages and then run gulp to get up and running, this is as simple as the following:

```bash
nvm use

```

This will make sure that you're using the right npm version. If it gives you an `No .nvmrc file found` error, then that's fine, forget about it.

Now we can install our front end packages:

```bash
npm install

```

This will install all front end dependencies.

Once this has finished, hopefully without any errors, we can compile everything.

```bash
gulp

```

or 

```bash
gulp watch

```

Will do this for you! Now visit your site again, and you should be golden 🏅



#### 6.2. Webpack

If you're not using the Gulp system, the chances are you're using the newer Webpack build system. This is a plus as it's **a lot** quicker than gulp.

To double check it, run the following commands:

```bash
cd assets
cat package.json | grep "start"

```

If you see something along the lines of ` "start": "npm run dev",` then you're using webpack 🎊

The awkward thing with the Webpack build is that some repos are managed by `yarn` and others by `npm` this is a bit annoying, but we'll get there.

From the assets directory, we need to install our frontend dependencies:



##### 6.2.1. NPM

Initially, lets try and install the dependencies with NPM.

Start by running the following command:

```bash
nvm use

```

This will make sure that you're using the right npm version. If it gives you an `No .nvmrc file found` error, then that's fine, forget about it.

We can then move onto our actual dependencies:

`npm install`

This will take a while, but should complete successfully, if it does, you'll see something along the lines of

```bash
added 2845 packages from 1464 contributors and audited 37582 packages in 185.815s

```

before it drops you back to your terminal, if it doesn't you'll get a ton of red error lines. If this is the case, speak to your lead dev to try and work through them.

If everything installed correctly, you can now run the following command and you should be golden!

```bash
yarn start

```





##### 6.2.2 Yarn

If NPM doesn't seem to be working for you, then it may be that someone has installed some packages with npm and others with yarn 😫 As we've already `npm install`'d then we can simply try and install any remaining packages with yarn. 

```bash
yarn

```

This command will instruct yarn to install any dependencies in the `yarn.lock` file. Fingers crossed this will install any of the other dependencies we need and then again we can try and run

```bash
yarn start

```

Hopefully, this will now be working!



### Common errors

Below are some of the sticking points I see quite often with Trellis setups:



#### 503 error when you try to access the site

As far as I can work out, this is an issue with nginx or PHP not starting when you run `vagrant up`. There are two ways around it I've found:

**SSH into your vagrant box.**

From the `trellis` directory where you run `vagrant up` run the `vagrant ssh` command. This will SSH you into the VM

**Restart PHP and nginx**

Once inside the VM, run the following two commands:

```bash
sudo service nginx restart
sudo service php7.2-fpm restart

```

If the PHP one doesn't play nice, you may need to adjust the name of it, try running the following to find the correct service name:

```bash
ls /etc/init.d/ | grep php

```


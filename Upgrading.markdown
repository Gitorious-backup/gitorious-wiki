# Upgrading Gitorious

If your version of Gitorious is really old, please start with this guide: [[LegacyUpgrade]] ("really old" = older than February 3rd 2009/92bb70a1).

If your version Gitorious is only a little old, refer to [[BundlerSetup]] ("a little old" = older than January 24th 2011/c44237f).

The changelog tool
===============

Since Gitorious 2.0.0 (see [[Versioning]]), Gitorious ship with a simple 
changelog tool. The tool consists of a single rake task that can tell you what version you are currently on and what versions are available to you. To use it, invoke the following command from the root of your installation:

    bundle exec rake changelog

The tool will give you a list of available version along with an arrow indicating your current version. If the arrow points to a green number, you're on top of things. If it points to a red number, it means that upgrades are available, and you are encouraged to stay up to date.

To display a changelog for changes between your current version and another version, run the tool with the `VERSION` environment variable set:

    bundle exec rake changelog VERSION=2.0.1

Before upgrading
==============

No matter how simple an upgrade may look, we strongly encourage you to always back up your database and repository data prior to upgrades. If nothing else, an upgrade is a good time to get some snapshots if you don't already have scheduled backups of your system. It's also a good idea to keep a copy of Gitorious configuration files outside the actual install.

Upgrading patch versions
====================

As per our [[Versioning]] manifesto, patch upgrades should be trivial in nature. The steps are simple:

    >> cd /path/to/gitorious
    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.x.y
    >> rake assets:clear

Note that `rake assets:clear` was not available prior to 2.1.0. 2.0.x can be directly upgraded to 2.1.0, see below.

As of 2.1.0 you also need these steps for any upgrade:

    >> git submodule init
    >> git submodule update
    >> rake db:migrate


Finally, restart all services. This includes the webserver, the poller and the git daemon. x/y depends on the actual version.

Upgrading customized installs
-----------------------------------------

If your Gitorious setup has changes, you may experience conflicts when pulling in changes. It is recommended to push your patched version of Gitorious somewhere, for example to itself, so you can easily clone Gitorious to a local machine, or somewhere else on the server to make sure that the patch will apply cleanly before attempting the upgrade.

Sample upgrade
-----------------------

Assuming you are on v2.0.0, run your app with [Phusion Passenger](http://www.modrails.com/) and run:

    >> bundle exec rake changelog
    Available versions
       v2.0.1              Next increment
    -> v2.0.0              First versioned version of Gitorious

Obviously, there's a new version in town, so let's see what it offers:

    >> bundle exec rake changelog VERSION=2.0.1
    Changes between v2.0.0 and v2.0.1:
    A longer description appears here

This looks good, so let's upgrade:

    git fetch git://gitorious.org/gitorious/mainline.git
    git merge v2.0.0
    rm public/stylesheets/all.css public/javascripts/all.js public/javascripts/capillary.js public/**/*/gts-*.*
    touch tmp/restart.txt

Upgrading from 2.0.x to 2.1
------------------------------------

You can upgrade directly from any 2.0.x version directly to 2.1.0. Start by reviewing changes:

    >> bundle exec rake changelog VERSION=2.1.0

If this looks good, back up everything (see above), and get started:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.1.0
    >> rake assets:clear

Gitorious now has submodules. Initialize and pull them:

    >> git submodule init
    >> git submodule update

Upgrade the database:

    >> rake db:migrate

Then restart your server (assuming you're using Passenger):

    >> touch tmp/restart.txt

Upgrading from 2.1.x to 2.2
------------------------------------

You can upgrade directly from any 2.1.x version directly to 2.2.0. Start by reviewing changes:

    >> bundle exec rake changelog VERSION=2.2.0

If this looks good, back up everything (see above), and get started:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.2.0
    >> rake assets:clear
    >> git submodule --init update

Upgrade the database:

    >> rake db:migrate

Then restart your server (assuming you're using Passenger):

    >> touch tmp/restart.txt

If you want to use the new [private repositories feature](https://gitorious.org/gitorious/pages/PrivateRepositories), set the `enable_private_repositories` setting to `true` in config/gitorious.yml. See the sample configuration in config/gitorious.sample.yml for more information.

Upgrading from 2.2.x to 2.3
------------------------------------

You can upgrade directly from any 2.2.x version directly to 2.3.0. Start by reviewing changes:

    >> bundle exec rake changelog VERSION=2.3.0

If this looks good, back up everything (see above), and get started:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.3.0
    >> rake assets:clear

Gitorious now has submodules. Initialize and pull them:

    >> git submodule init
    >> git submodule update

Upgrade the database:

    >> rake db:migrate

Then restart your server (assuming you're using Passenger):

    >> touch tmp/restart.txt

Upgrading from 2.3.0 to 2.3.1
----------------------------------------

To upgrade from version 2.3.0 to 2.3.1, follow these steps:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.3.1
    >> bundle install
    >> rake assets:clear
    >> touch tmp/restart.txt

Upgrading from 2.3.2 to 2.4.1
----------------------------------------

To upgrade from version 2.3.2 to 2.4.1, follow these steps:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.4.1
    >> bundle install
    >> rake assets:clear
    >> rake db:migrate
    >> rake ts:rebuild
    >> touch tmp/restart.txt

Upgrading patch releases in the 2.4 series
-----------------------------------------------------------

Due to our use of git-flow there have been a few patch releases in the 2.4 series. To upgrade between these:

    >> git fetch git://gitorious.org/gitorious/mainline.git
    >> git merge v2.4.x
    >> bundle install
    >> rake assets:clear
    >> rake db:migrate
    >> rake ts:rebuild
    >> touch tmp/restart.txt

Upgrading from version 2.4.x to latest 3.0.x
-------------------------------------------------

To upgrade from 2.4.x to latest 3.0.x, follow these steps:

    >> git clone https://git.gitorious.org/gitorious/ce-installer.git && cd ce-installer
    >> sudo ./upgrade.sh

Upgrading patch releases in the 3.0.x series
---------------------------------------------------------------

To upgrade from any 3.0.x version to latest 3.0.x (currently 3.0.4), follow these steps:

    >> git clone https://git.gitorious.org/gitorious/ce-installer.git && cd ce-installer
    >> sudo ./upgrade.sh

Upgrading from 3.0.x to development version
---------------------------------------------------------------------

It is possible to install the latest version directly from the development branch,  follow these steps:

    >> git clone https://git.gitorious.org/gitorious/ce-installer.git && cd ce-installer
    >> sudo GITORIOUS_VERSION="origin/master" ./upgrade.sh
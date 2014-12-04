# Upgrading Gitorious

If your version of Gitorious is really old (older than February 3rd 2009/92bb70a1), please start with this guide: [[LegacyUpgrade]].

If your version Gitorious is only a little old (older than January 24th 2011/c44237f), refer to [[BundlerSetup]].

## The changelog tool

Since Gitorious 2.0.0 (see [[Versioning]]), Gitorious ship with a simple 
changelog tool. The tool consists of a single rake task that can tell you what version you are currently on and what versions are available to you. To use it, invoke the following command from the root of your installation:

    bundle exec rake changelog

The tool will give you a list of available version along with an arrow indicating your current version. If the arrow points to a green number, you're on top of things. If it points to a red number, it means that upgrades are available, and you are encouraged to stay up to date.

To display a changelog for changes between your current version and another version, run the tool with the `VERSION` environment variable set:

    bundle exec rake changelog VERSION=2.0.1

## Before upgrading

No matter how simple an upgrade may look, we strongly encourage you to always back up your database and repository data prior to upgrades. If nothing else, an upgrade is a good time to get some snapshots if you don't already have scheduled backups of your system. It's also a good idea to keep a copy of Gitorious configuration files outside the actual install.

### Upgrading customized installs

If your Gitorious setup has changes, you may experience conflicts when pulling in changes. It is recommended to push your patched version of Gitorious somewhere, for example to itself, so you can easily clone Gitorious to a local machine, or somewhere else on the server to make sure that the patch will apply cleanly before attempting the upgrade.

## Upgrading

### From 2.0.x to 2.1

You can upgrade directly from any 2.0.x version directly to 2.1.0. Start by reviewing changes:

    bundle exec rake changelog VERSION=2.1.0

If this looks good, back up everything (see above), and get started:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.1.0
    rake assets:clear
    git submodule --init update

Upgrade the database:

    rake db:migrate

Then restart your server (assuming you're using Passenger):

    touch tmp/restart.txt

### From 2.1.x to 2.2

You can upgrade directly from any 2.1.x version directly to 2.2.0.

First back up everything (see above), and then get started:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.2.0
    rake assets:clear
    git submodule --init update

Upgrade the database:

    rake db:migrate

Then restart your server (assuming you're using Passenger):

    touch tmp/restart.txt

If you want to use the new [private repositories feature](https://gitorious.org/gitorious/pages/PrivateRepositories), set the `enable_private_repositories` setting to `true` in config/gitorious.yml. See the sample configuration in config/gitorious.sample.yml for more information.

### From 2.2.x to 2.3

You can upgrade directly from any 2.2.x version directly to 2.3.0.

First back up everything (see above), and then get started:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.3.0
    rake assets:clear
    git submodule --init update

Upgrade the database:

    rake db:migrate

Then restart your server (assuming you're using Passenger):

    touch tmp/restart.txt

### From 2.3.0 to 2.3.1

To upgrade from version 2.3.0 to 2.3.1, follow these steps:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.3.1
    bundle install
    rake assets:clear
    git submodule --init update
    touch tmp/restart.txt

### From 2.3.2 to 2.4.1

To upgrade from version 2.3.2 to 2.4.1, follow these steps:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.4.1
    bundle install
    rake assets:clear
    git submodule --init update
    rake db:migrate
    rake ts:rebuild
    touch tmp/restart.txt

### From 2.4.x to 2.4.12 (latest 2.4 patch release)

Due to our use of git-flow there have been a few patch releases in the 2.4 series. To upgrade between these:

    git fetch https://gitorious.org/gitorious/mainline.git
    git merge v2.4.x
    bundle install
    rake assets:clear
    git submodule --init update
    rake db:migrate
    rake ts:rebuild
    touch tmp/restart.txt

### From 2.4.x / 3.x to latest 3.x

To upgrade from Gitorious 2.4.x or 3.x to latest stable release (3.2 as of now), follow these steps:

    git clone https://gitorious.org/gitorious/ce-installer.git && cd ce-installer
    sudo ./upgrade.sh

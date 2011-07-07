# Upgrading Gitorious

If your version of Gitorious is really old, please start with this guide: [[LegacayUpgrade]] ("really old" = older than February 3rd 2009/92bb70a1).

If your version Gitorious is only a little old, refer to [[BundlerSetup]] ("a little old" = older than January 24th/c44237f).

The changelog tool
===============

As of Gitorious 2.0.0 (see [[Versioning]]), Gitorious ship with a simple changelog tool. The tool consists of a single rake task that can tell you what version you are currently on and what versions are available to you. To use it, invoke the following command from the root of your installation:

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

    cd /path/to/gitorious
    git fetch git://gitorious.org/gitorious/mainline.git
    git merge v2.x.y
    rm public/stylesheets/all.css public/javascripts/all.js

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
    git merge v2.0
    rm public/stylesheets/all.css public/javascripts/all.js
    touch tmp/restart.txt

Upgrading minor versions
====================

Minor versions will come with individual upgrade notes. Check back here when the time for 2.1 comes.
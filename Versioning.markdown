Gitorious Versioning
================

As of July 7th 2011, Gitorious has a versioning scheme, and the initial version is 2.0.0. This version is chosen arbitrarily, but we chose 2 over 1 to reflect the fact that Gitorious has been through a few major changes since its inception, and we've been around for almost 4 years.

Below you can read about the types of changes and what they demand from someone upgrading Gitorious. The changes are listed in order of most to least frequent. Releases are available as tags of the form "vx.y.z" in the [Gitorious mainline repository](https://gitorious.org/gitorious/mainline/trees/v2.0.0).

Risks of upgrading
===============

We will do our best to keep releases stable. Every release will be run in production on gitorious.org prior to being tagged.

Patch versions
============

Patch versions indicate changes with minor impact for installs. In general, the rools for patch versions are:

* Should not require database migrations
* Should not require configuration changes or additions
* Should not require changes in CSS (e.g. by changing existing markup)

In general: If you have a local install, you should be able to safely upgrade patch releases without changing anything, even if you have local CSS adjustments.

Because of these restrictions, patch upgrades should be considered fairly trivial and of minimal risk. Refer to [[Upgrading]] for information on checking your current version and upgrading.

Minor versions
============

Minor versions are bigger changes or changes that require action on your part other than simply pulling from Git and restarting your app. These changes may include:

* Database migrations
* Required configuration changes and/or additions
* Substantial changes/additions to the view
* API changes in models and key lib classes

Each minor version will be published with a separate upgrade guide here in the wiki. The risk should still be minimal 

Major versions
============

Major versions will presumably occur infrequently, and there is no specific rules to trigger an upgrade. Our rool of thumb right now is that if a new version of Gitorious requires an entirely new installation guide of its own, it's probably a major release. However, we may also up the major version number due to the addition of substantial new features, redesigns and whatnot.

How often/when will Gitorious update its version?
======================================

Releases will be made on an irregular schedule, and will coincide with gitorious.org deployments, but not in a 1:1 ratio. Occasionally we deploy Gitorious a number of times throughout a single week, but do not expect more than one version per week. Follow this page, or better - the [mailing list](http://groups.google.com/group/gitorious) for updates.

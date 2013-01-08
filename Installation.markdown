Active development happens on the git branch ":master", and we will try to keep it stable. However, the tags will receive more testing, and in general should be more reliable as a mechanism to upgrade local installs. git master should be considered "Gitorious beta" (generally stable, may have occasional issues, incomplete features a.s.o), while tags provide stable releases.

For a Chef based automated install for Debian Squeeze or recent Ubuntu you can try:

* [this recent cookbook](http://community.opscode.com/cookbooks/gitorious) available on [github](http://github.com/brugidou/gitorious-cookbook) for single server install
* or [these instructions](https://github.com/rosenfeld/gitorious-cookbooks).
* There is [another one](https://github.com/fnichol/chef-gitorious) using RVM for installing Ruby
* and some (a little bit) outdated [instructions](http://rosenfeld.heroku.com/en/articles/2011-03-06-installing-gitorious-has-never-been-so-easy) on how to set-up it using Chef-solo.

## See Also

[[Upgrading]]

* [[UbuntuInstallation]] - Step by step instructions for installing Gitorious under Ubuntu 9.04
* [[Gentoo_Installation]] - The portage way.
* [[Rhel_Installation]] - Installation guide for RHEL 5 by some kickass team called Verso.
* [[DebianLennyInstallation]] - Step by step instructions for install Gitorious under Debian Lenny 5.0.8
* [[DebianSqueezeInstallation]] - Step by step instructions for install Gitorious under Debian Squeeze 6.0.0
* [[DebianWheezyInstallation]] - Step by step instructions for install Gitorious under Debian Wheezy 7.0
* [OSXInstallation](http://www.bluequartz.net/projects/EIM_Segmentation/SCMService/html/gitorious___o_s_x__setup.html) - Step by step instructions for install of Gitorious under OSX 10.6
* [SolarisInstallation](http://www.cloudcomp.ch/2013/01/installing-gitorious-on-solaris-sparc/) - Hints and Instructions for install of Gitorious under Oracle Solaris 11.1

After installation, see [[Configuration]].

[DISCONTINUED - TODO: should we remove this reference?] If you want a automated installation script for gitorious in ubuntu you can try [this one](http://github.com/marcelomurad/rails-env-install) written by [Marcelo Murad](http://marcelomurad.com).
Installing and Setting Up Gitorious on RHEL 5
=============================================

** Note: Parts of the contents of this page are obsolete.** [Here is a page that takes you through installation on 64 bit Centos systems](http://famousphil.com/blog/2011/06/installing-gitorious-on-centos-5-6-x64/), written in June 2011.

Marko Peltola
2010-06-15 (Updated: 2010-08-11)

Verso Project
Department of Mathematical Information Technology
University of Jyväskylä

[Gitorious](http://gitorious.org/) is an open source application originally
developed for hosting open source projects in git repositories. This document
describes how to install Gitorious on Red Hat Enterprise 5 (RHEL 5) server.

This guide also covers how to install the services Gitorious is dependent on,
including:

  * Web server (Apache)
  * Database server (MySQL)
  * Ruby (Enterprise Edition)


The guide is based on:

  * [Setting up Gitorious on your own server](http://cjohansen.no/en/ruby/setting_up_gitorious_on_your_own_server)
  * [rough notes for installing gitorious on fedora 11](http://groups.google.com/group/gitorious/browse_thread/thread/72cf30f5ea823113?pli=1)


The installation guide is tested with customized version of Gitorious named
YouSource, version 1f2c62 (Git hash). Though the installation process is the
same on the both versions.


All the commands are meant to run as root user, if not mentioned otherwise.


Install Required Packages
-------------------------

Gitorious needs a lot of software:

    yum install -y git git-svn pcre pcre-devel zlib zlib-devel sendmail ;
    yum -y groupinstall "Development Libraries" "Development Tools" ;
    yum install -y libssh2 libssh2-devel openssh openssh-server \
        memcached libyaml libyaml-devel ImageMagick ImageMagick-devel  \
        apr-devel uuid java-1.6.0-openjdk readline-devel glibc-devel \
        openssl-devel gcc-c++ gcc-c++ zlib-devel openssl-devel readline-devel \
        sphinx


Install Ruby and Rubygems
-------------------------

Gitorious runs on a variety of Rubys (>= 1.8.6). If you want to use the basic
Ruby, you can install it with:

    yum install -y ruby ruby-devel rubygems


If you want to install Ruby Enterprise Edition, you have to download the source
tarball. The source tarball contains a cross-platform installer. This guide is
based on Ruby Enterprise so I recommend to use it.

    wget http://rubyforge.org/frs/download.php/68719/ruby-enterprise-1.8.7-2010.01.tar.gz
    tar xzvf ruby-enterprise-1.8.7-2010.01.tar.gz
    ./ruby-enterprise-1.8.7-2010.01/installer


Install Required Gems
---------------------

Gitorious requires a lot of gem packages.

    gem update --system
    gem install --no-ri --no-rdoc rails mongrel mime-types \
        textpow chronic ruby-hmac daemons mime-types oniguruma passenger \
        textpow chronic BlueCloth  ruby-yadis ruby-openid geoip ultrasphinx \
        rspec rspec-rails RedCloth echoe hoe diff-lcs stompserver json mysql

Important! Latest version of rack wasn't compatible with our testing
environment, you may need to use version 1.0.1 instead.

    gem install --no-ri --no-rdoc -v 1.0.1 rack



Install and Configure MySQL
---------------------------

Gitorious should work with any SQL server, but it's only tested with MySQL.

* Install MySQL
      yum install mysql mysql-server mysql-libs mysql-devel

* Start the service
      service mysqld start

* Set a password for root user. Replace `<password>` with your own password.
      mysqladmin -u root password '<password>'



Set Up git User
---------------

Gitorious needs a specific user which runs the services. Here it's named git.
Gitorious uses SSH keys in authentication in Git push. The git user needs to
have a home directory, where the SSH keys are saved.

    adduser --create-home git


Because `git push` is authenticated via git user, the git user must have a ssh
access to the server. So add git user to `AllowGroups` in `/etc/ssh/sshd_config`.


Clone the Gitorious Repository
------------------------------

Go to the directory where you want to download the application, for example to
`/var/www`. Use git to download Gitorious.

    git clone git://gitorious.org/gitorious/mainline.git gitorious

Or if you want to install customized version of Gitorious named YouSource, 
(currently only available inside the Jyväskylä University network):

    git clone git://versotest.it.jyu.fi/verso/gitorious.git

Make a symbolic link of gitorious script to `/usr/local/bin/`.

    ln -s /var/www/gitorious/script/gitorious /usr/local/bin/gitorious


Create the Directories for Repositories and Tarballs
----------------------------------------------------

Make the directories, where the git repositories and tarballs are saved.

    mkdir /var/git
    mkdir /var/git/repositories
    mkdir /var/git/tarballs
    mkdir /var/git/tarball-work

Change the owner of these and the application directories to git user.

    chown -R git:git /var/git /var/www/gitorious    


Install a Web Server
--------------------

Gitorious is a web application, so you need a web server. Here we install
Apache, but you can use some other web server as well.

    yum install -y httpd httpd-devel


Install also the mod_xsendfile module to get source tarball downloading working.

    wget http://tn123.ath.cx/mod_xsendfile/mod_xsendfile.c
    apxs -cia mod_xsendfile.c


Install and Configure Passenger
-------------------------------

Install Phusion Passenger module to simplify Rails application deployment.

    /opt/ruby-enterprise/bin/passenger-install-apache2-module

Follow the instructions as provided by the installation script.

Edit Apache configuration file `/etc/httpd/conf/httpd.conf` and add the lines
the installation script recommend, something like this:

    LoadModule passenger_module /opt/ruby-enterprise/lib/ruby/gems/1.8/gems/passenger-2.2.11/ext/apache2/mod_passenger.so
    PassengerRoot /opt/ruby-enterprise/lib/ruby/gems/1.8/gems/passenger-2.2.11
    PassengerRuby /opt/ruby-enterprise/bin/ruby


In the same file set DocumentRoot to `<your-gitorious-root>/public`, for example
`/var/www/gitorious/public`, and set your ServerName (ours is versotest.it.jyu.fi).

   <VirtualHost *:80>
      ServerName versotest.it.jyu.fi
      DocumentRoot /var/www/gitorious/public

      # Enable X-SendFile for Gitorious repo archiving to work
      XSendFile on
      XSendFileAllowAbove on
      
      <Directory /var/www/gitorious/public>
         AllowOverride all
         Options -MultiViews
      </Directory>
   </VirtualHost>

   <VirtualHost *:443>
      ServerName versotest.it.jyu.fi
      DocumentRoot /var/www/gitorious/public
      
      <Directory /var/www/gitorious/public>
         AllowOverride all
         Options -MultiViews
      </Directory>
   </VirtualHost>

Restart Apache and after that Passenger should recognize the application and
start it when accessing the web page through a web browser.


Set Up Gitorious
================

The git user runs all the Gitorious services. So from now on use git user:

    sudo su git


Add these lines in the end of the git user's `~/.bashrc` file so that git user
can run ruby programs. Change the directory to match your Ruby installation
directory.

    export RUBY_HOME=/opt/ruby-enterprise
    export GEM_HOME=$RUBY_HOME/lib/ruby/gems/1.8/gems
    export PATH=$RUBY_HOME/bin:$PATH

Load new paths as a git user:

    source ~/.bashrc


Make authorized_keys File
-------------------------

When someone tries to push to a repository through Gitorious, the user will be
looked up in the git user's `~/.ssh/authorized_keys`. If the user is found here,
the ssh connection is handled by the git-daemon process. We have to create this
file, but Gitorious will maintain it for us. 

Make sure the permissions on `authorized_keys` are 600 and `~/.ssh` are 700.

    mkdir ~/.ssh
    chmod 700 ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/authorized_keys


Configuration
-------------

Next go to your Gitorious root dir (for example `cd /var/www/gitorious`) and
make a directory for pid files.

    mkdir -p tmp/pids

Make all the Gitorious scripts executable.

    chmod ug+x script/*


Copy the configuration sample files and edit the settings. The files includes
good instructions.

    cp config/database.sample.yml config/database.yml
    cp config/gitorious.sample.yml config/gitorious.yml
    cp config/broker.yml.example config/broker.yml


If you installed customized YouSource, you have to edit these files too:

    cp config/email.yml.example config/email.yml
    cp script/ldap_settings.py.template script/ldap_settings.py


Install the remaining gems:

    sudo rake gems:install 


Create the database and schema:

    rake db:create RAILS_ENV=production
    rake db:migrate RAILS_ENV=production


Build the search index and start the search daemon:

    rake ultrasphinx:configure RAILS_ENV=production
    rake ultrasphinx:index RAILS_ENV=production
    rake ultrasphinx:daemon:start RAILS_ENV=production


Schedule Sphinx search engine to index the site automatically. Add to the
crontab, using `crontab -e`, the following line:

    * * * * * cd /var/www/gitorious && /opt/ruby-enterprise/bin/rake ultrasphinx:index RAILS_ENV=production


Scripts
-------

Copy `git-daemon` and `git-ultrasphinx` from `doc/templates/centos` to
`/etc/init.d/` as root user.

You have to create `gitorious-poller` and `stomp` scripts on your own.

gitorious-poller Example File:
------------------------------

    #!/bin/sh
    #
    # poller       Startup script for Gitorious's poller
    #
    # chkconfig: - 86 15
    # description: Gitorious's poller script is simple worker that polls \
    #              tasks from stomp server queue and executes them.
    # processname: poller

    /bin/su - git -c "cd /var/www/gitorious; RAILS_ENV=production ./script/poller $@"


stomp Example File:
-------------------

    #!/bin/sh
    #
    # stomp        Startup script for stomp server 
    #
    # chkconfig: - 85 15
    # description: Stomp server is simple task queue server that \
    #              uses stomp protocol.     
    # processname: stomp
    # config: /etc/stompserver.conf

    /bin/su - git -c "cd /var/www/gitorious; RAILS_ENV=production; stompserver -C /etc/stompserver.conf $@"


Start the Services
------------------

Go to `/etc/init.d/` and give the scipts execute permissions:

    sudo chmod 755 git-daemon git-ultrasphinx gitorious-poller stomp


Add services to start automatically on boot:

    chkconfig --add stomp
    chkconfig --add git-daemon
    chkconfig --add gitorious-poller
    chkconfig --add git-ultrasphinx
    chkconfig --add memcached

    chkconfig stomp on
    chkconfig git-daemon on
    chkconfig gitorious-poller on
    chkconfig git-ultrasphinx on
    chkconfig memcached on


Now you can start the services:

    service git-ultrasphinx start
    service stomp start
    service memcached start
    service gitorious-poller start
    service git-daemon start
    service httpd restart


Now you should have Gitorious up and running! Go to your web page through a
browser and start using Gitorious.

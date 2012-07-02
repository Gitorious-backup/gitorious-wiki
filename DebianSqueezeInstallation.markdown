This article is a rip off [Setting up gitorious on your own server](http://cjohansen.no/en/ruby/setting_up_gitorious_on_your_own_server) and [[UbuntuInstallation]].  Many thanks to the authors of those documents for the work they've put in.  I've added only slight modifications that are specific to Debian Squeeze.

The following steps should take you from a freshly installed copy of Debian Squeeze 6.0.0 to a fully functioning Gitorious server.

# Install packages
    # If you use MariaDB instead of MySQL, s/mysql/mariadb/g
    sudo aptitude update
    sudo aptitude install build-essential zlib1g-dev libcurl4-openssl-dev \
                apache2 mysql-server mysql-client apg geoip-bin libgeoip1 \
                libgeoip-dev libpcre3 libpcre3-dev zlib1g zlib1g-dev libyaml-dev \
                libmysqlclient-dev apache2-dev libonig-dev zip unzip memcached \
                git-core git-svn git-doc git-cvs libreadline-dev openjdk-6-jre \
                sqlite3 libsqlite3-dev libmagick++3 libmagick++-dev libapache2-mod-xsendfile \
                libxslt1-dev libreadline5


# Install Ruby enterprise
I'm already using Phusion's Ruby Enterprise Edition (REE) + passenger combination for other web services on this host. The following is for the latest release (currently 1.8.7-2011.01).  I use the default install location (`/opt`).  Linking gem, rake and ruby into `/usr/local/bin` is optional.  I do find that it prevents having to set PATH in a number of different scripts seperately.  Please update me if there's a better way to do this.
    wget http://rubyenterpriseedition.googlecode.com/files/ruby-enterprise-1.8.7-2011.03.tar.gz
    tar zxf ruby-enterprise-1.8.7-2011.03.tar.gz
    sudo ruby-enterprise-1.8.7-2011.03/installer
    cd /opt && sudo ln -s ruby-enterprise-1.8.7-2011.03/ ruby-enterprise
    sudo ln -s /opt/ruby-enterprise/bin/ruby /opt/ruby-enterprise/bin/rake /opt/ruby-enterprise/bin/gem /usr/local/bin

# Install Sphinx
Compile latest version (currently 2.0.3)
    wget http://sphinxsearch.com/files/sphinx-2.0.3-release.tar.gz
    tar zxf sphinx-2.0.3-release.tar.gz && cd sphinx-2.0.3
    ./configure --prefix=/usr/local/sphinx-2.0.3
    make && sudo make install
    cd /usr/local && sudo ln -s sphinx-2.0.3 sphinx
    sudo ln -s /usr/local/sphinx/bin/* /usr/local/bin

# Install Apache ActiveMQ
Download the latest [binaries](http://www.apache.org/dyn/closer.cgi?path=%2Factivemq%2Fapache-activemq%2F5.5.1%2Fapache-activemq-5.5.1-bin.tar.gz) (currently 5.5.1):
    tar zxf apache-activemq-5.5.1-bin.tar.gz
    sudo mv apache-activemq-5.5.1 /usr/local
    cd /usr/local && sudo ln -s apache-activemq-5.5.1 apache-activemq && cd apache-activemq
    sudo bin/activemq setup /etc/default/activemq && sudo chmod 600 /etc/default/activemq
    sudo adduser --system --no-create-home --home /usr/local/apache-activemq --shell /bin/bash activemq
    sudo chown -R activemq:nogroup /usr/local/apache-activemq-5.5.1

Edit `/etc/default/activemq` and add to the beginning of the file:
    ACTIVEMQ_HOME=/usr/local/apache-activemq

And change the `ACTIVEMQ_USER=` line to read:
    ACTIVEMQ_USER="activemq"

Edit `/usr/local/apache-activemq/conf/activemq.xml`.  Comment out the existing transportConnector line and add the stomp protocol.  The section should look like the following:
        <transportConnectors>
            <!--
              <transportConnector name="openwire" uri="tcp://0.0.0.0:61616"/>
            -->
            <transportConnector name="stomp" uri="stomp://localhost:61613"/>
        </transportConnectors>

# Fetch Gitorious
    sudo git clone git://gitorious.org/gitorious/mainline.git /var/www/gitorious
-- or --
    sudo git clone http://git.gitorious.org/gitorious/mainline.git /var/www/gitorious

With release 2.0.0 gitorious began tagging releases as outlined [here](https://gitorious.org/gitorious/pages/Versioning).  This provides for cleaner and hopefully safer transitions between versions.  For each new version, I basically checkout a new branch based off a tag.  That way, I can make changes or merge into my own branch.
 git checkout -b my2.1.1 v2.1.1

# Install Ruby gems
For this, I gave my git user temporary sudo access.  You could just execute this as root as well.
    cd /var/www/gitorious &&  /opt/ruby-enterprise/bin/bundle install

# Configure services
* Copy Sphinx and git-daemon scripts from `gitorious/doc/templates/ubuntu` to `/etc/init.d`; change Ruby interpreter in git-daemon if you used a non-default location.  Also, you'll want to change the `Provides` line in these scripts to something without spaces.  I've used `gitorious-git-daemon` and `gitorious-ultrasphinx`.
        cd /var/www/gitorious/doc/templates/ubuntu
        sudo cp git-daemon git-ultrasphinx /etc/init.d
* Create poller init script (see Files below)
        sudo vi /etc/init.d/git-poller
* Create activemq init script (see Files below)
        sudo vi /etc/init.d/activemq
* Copy gitorious-logrotate from `gitorious/doc/templates/ubuntu` to `/etc/logrotate.d/gitorious`
        sudo cp gitorious-logrotate /etc/logrotate.d/gitorious
* Make init scripts run at startup
        sudo chmod 755 /etc/init.d/git-ultrasphinx /etc/init.d/git-daemon /etc/init.d/git-poller /etc/init.d/activemq
        sudo insserv /etc/init.d/git-ultrasphinx /etc/init.d/git-daemon /etc/init.d/git-poller /etc/init.d/activemq
* Link the gitorious script into `/usr/local/bin`
        sudo ln -s /var/www/gitorious/script/gitorious /usr/local/bin/

# Configure Apache2
* Install Passenger
        sudo /opt/ruby-enterprise/bin/passenger-install-apache2-module
        if you get an error make this before to continue ( sudo /opt/ruby-enterprise/bin/gem install passenger)
* Create `/etc/apache2/mods-available/passenger.load`:
        LoadModule passenger_module /opt/ruby-enterprise/lib/ruby/gems/1.8/gems/passenger-3.0.5/ext/apache2/mod_passenger.so
        PassengerRoot /opt/ruby-enterprise/lib/ruby/gems/1.8/gems/passenger-3.0.5
        PassengerRuby /opt/ruby-enterprise/bin/ruby
* Enable needed modules
        sudo a2enmod passenger rewrite ssl xsendfile
* Create /etc/apache2/sites-available/gitorious (see Files below)
* Create /etc/apache2/sites-available/gitorious-ssl (see Files below)
* Disable the default sites and enable the Gitorious sites
        sudo a2dissite default default-ssl
        sudo a2ensite gitorious gitorious-ssl
* Restart Apache
        sudo /etc/init.d/apache2 restart

# Configure Gitorious
* Add a git user to MySQL:
        mysql -u root -p
            create database gitorious_production;
            grant all privileges on gitorious_production.* to 'git'@'localhost' identified by 'YOUR_PASSWORD';
* Run the following
        sudo adduser --system --home /var/www/gitorious --no-create-home --group --shell /bin/bash git
        sudo chown -R git:git /var/www/gitorious
        sudo su - git
        git submodule init
        git submodule update
        mkdir .ssh
        touch .ssh/authorized_keys
        chmod -R go-rwx .ssh
        mkdir tmp/pids repositories tarballs  tarballs-work
        cp config/database.sample.yml config/database.yml
        cp config/gitorious.sample.yml config/gitorious.yml
        cp config/broker.yml.example config/broker.yml

* Edit `config/database.yml`: Remove every section but production
  * `username` should be `git`
  * `password` should be the password indicated in 2 before sections
* Edit `config/gitorious.yml`:  Remove every section but production
  * `repository_base_path` should be something like `/var/www/gitorious/repositories`
  * `gitorious_client_port` should be 80
  * `gitorious_host` needs to be the exact hostname that clients will use (cookies get messed up otherwise)
  * `archive_cache_dir` should be something like `/var/www/gitorious/tarballs`
  * `archive_work_dir` should be something like `/var/www/gitorious/tarballs-work`
  * `hide_http_clone_urls` should be `true` (they require extra unknown setup to work)
  * `is_gitorious_dot_org` should be `false`
* Run the following
        
    export RAILS_ENV=production
    bundle exec rake db:create
    bundle exec rake db:migrate
    export PATH=/usr/local/sphinx/bin:$PATH
    bundle exec rake ultrasphinx:bootstrap

    crontab -e
        PATH=/usr/local/sphinx/bin:/usr/bin:/bin
        * * * * * cd /var/www/gitorious && /opt/ruby-enterprise/bin/bundle exec /opt/ruby-enterprise/bin/rake ultrasphinx:index RAILS_ENV=production

Also see [[Configuration]].

# Finish
    sudo /etc/init.d/git-daemon start
    sudo /etc/init.d/git-ultrasphinx start
    sudo /etc/init.d/activemq start
    sudo /etc/init.d/git-poller start
    sudo /etc/init.d/apache2 restart

* Create a Admin User as User 'git':
       env RAILS_ENV=production script/create_admin

# Console User Administration
* Delete a User
       env RAILS_ENV=production script/console
       > user = User.find_by_login "Username"
       > user.destroy
* Create Roles
        env RAILS_ENV=production script/console
        > Role.create!(:name => "Member", :kind => Role::KIND_MEMBER)
        > Role.create!(:name => "Administrator", :kind => Role::KIND_ADMIN)

# Files

/etc/apache2/sites-available/gitorious
    <VirtualHost *:80>
            ServerAdmin webmaster@localhost

            DocumentRoot /var/www/gitorious/public
            
            ErrorLog /var/log/apache2/gitorious-error.log
            # Possible values include: debug, info, notice, warn, error, crit,
            # alert, emerg.
            LogLevel warn

            CustomLog /var/log/apache2/gitorious-access.log combined

            <IfModule mod_xsendfile.c>
                    XSendFile on
                    XSendFileAllowAbove on
            </IfModule>


    </VirtualHost>


/etc/apache2/sites-available/gitorious-ssl
    <IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost

        DocumentRoot /var/www/gitorious/public

        ErrorLog /var/log/apache2/gitorious-error.log

        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn

        CustomLog /var/log/apache2/gitorious-ssl_access.log combined

        SSLEngine on

        SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem
        SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key

        BrowserMatch ".*MSIE.*" \
                nokeepalive ssl-unclean-shutdown \
                downgrade-1.0 force-response-1.0

        <IfModule mod_xsendfile.c>
                XSendFile on
                XSendFileAllowAbove on
        </IfModule>


    </VirtualHost>
    </IfModule>

/etc/init.d/activemq
    #!/bin/sh -e
    ### BEGIN INIT INFO
    # Provides:          activemq
    # Required-Start:    $network $local_fs
    # Required-Stop:     $network $local_fs
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Start/stop activemq for stomp services.
    ### END INIT INFO
    
    /usr/local/apache-activemq/bin/activemq "$@"

/etc/init.d/git-poller
    #!/bin/sh
    # Start/stop the git poller
    #
    ### BEGIN INIT INFO
    # Provides:          git-poller
    # Required-Start:    activemq mysql
    # Required-Stop:  activemq mysql
    # Default-Start:     2 3 4 5
    # Default-Stop:      1
    # Short-Description: Gitorious poller
    # Description:       Gitorious poller
    ### END INIT INFO
     
    /bin/su - git -c "cd /var/www/gitorious;RAILS_ENV=production script/poller $@"
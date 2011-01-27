This article is a rip off [Setting up gitorious on your own server](http://cjohansen.no/en/ruby/setting_up_gitorious_on_your_own_server) and [[UbuntuInstallation]].  Many thanks to the authors of those documents for the work they've put in.  I've added only slight modifications that are specific to Debian Lenny.

The release of squeeze should make things a little easier.  There would be no need to compile ImageMagick or use the SQLite3 backports.

The following steps should take you from a freshly installed copy of Debian Lenny 5.0.8 to a fully functioning Gitorious server.

# Install packages
    sudo aptitude update  
    sudo aptitude install build-essential zlib1g-dev libcurl4-openssl-dev \
                apache2 mysql-server mysql-client apg geoip-bin libgeoip1 \
                libgeoip-dev libpcre3 libpcre3-dev zlib1g zlib1g-dev libyaml-dev \
                libmysqlclient15-dev apache2-dev libonig-dev zip unzip memcached \
                git-core git-svn git-doc git-cvs libreadline-dev openjdk-6-jre

# Install sqlite
The version in Lenny 5.0.8 is too old for the current versions of certain ruby gems.  We'll need to install from backports.  Alternatively, you may trying compiling it on your own.

Add the following to your `/etc/apt/sources.list`
    deb http://backports.debian.org/debian-backports lenny-backports main

Add the following to `/etc/apt/preferences` (this file doesn't exist by default).  This is so that updates get applied properly:
    Package: *
        Pin: release a=lenny-backports
        Pin-Priority: 200

Install sqlite and dev files (3.7.3 at time of writing):
    sudo aptitude update
    sudo aptitude -t lenny-backports install sqlite3 libsqlite3-dev

# Install ImageMagick
The version in Lenny 5.0.8 is too old for the current rmagick gem (currently 2.13.1).  We'll compile from the latest source (currently 6.6.7).
    wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick-6.6.7-2.tar.bz2
    tar jxf ImageMagick-6.6.7-2.tar.bz2 && cd ImageMagick-6.6.7-2
    ./configure --prefix=/usr/local/ImageMagick-6.6.7.2
    make && sudo make install
    cd /usr/local && sudo ln -s ImageMagick-6.6.7.2 ImageMagick
  
Create `/etc/ld.so.conf.d/ImageMagick.conf` with the following line:
    /usr/local/ImageMagick/lib

Run ldconfig
    sudo ldconfig

# Install Ruby enterprise
I'm already using Phusion's Ruby Enterprise Edition (REE) + passenger combination for other web services on this host. The following is for the latest release (currently 1.8.7-2010.02).  I use the default install location (`/opt`).  Linking gem, rake and ruby into `/usr/local/bin` is optional.  I do find that it prevents having to set PATH in a number of different scripts seperately.  Please update me if there's a better way to do this.
    wget http://rubyforge.org/frs/download.php/71096/ruby-enterprise-1.8.7-2010.02.tar.gz
    tar zxf ruby-enterprise-1.8.7-2010.02.tar.gz
    sudo ruby-enterprise-1.8.7-2010.02/installer
    cd /opt && sudo ln -s ruby-enterprise-1.8.7-2010.02 ruby-enterprise
    sudo ln -s /opt/ruby-enterprise/bin/ruby /opt/ruby-enterprise/bin/rake /opt/ruby-enterprise/bin/gem /usr/local/bin 

# Install Ruby gems
    sudo gem install -b --no-ri --no-rdoc chronic geoip daemons hoe \
        echoe ruby-yadis ruby-openid mime-types diff-lcs json ruby-hmac
    sudo PATH=/usr/local/ImageMagick/bin:$PATH gem install -b --no-ri --no-rdoc rmagick
    sudo gem install -b --no-ri --no-rdoc -v 1.0.1 rack
    sudo gem install -b --no-ri --no-rdoc -v 1.3.1.1 rdiscount
    sudo gem install -b --no-ri --no-rdoc -v 1.1 stomp
    sudo gem uninstall -I i18n    # latest versions conflict with gitorious

# Install Sphinx
Compile latest version (currently 0.9.9)
    wget http://sphinxsearch.com/files/sphinx-0.9.9.tar.gz
    tar zxf sphinx-0.9.9.tar.gz && cd sphinx-0.9.9
    ./configure --prefix=/usr/local/sphinx-0.9.9
    make && sudo make install
    cd /usr/local && sudo ln -s sphinx-0.9.9 sphinx
    sudo ln -s /usr/local/sphinx/bin/* /usr/local/bin

# Install Apache ActiveMQ
Download the latest [binaries](http://www.apache.org/dyn/closer.cgi?path=/activemq/apache-activemq/5.4.2/apache-activemq-5.4.2-bin.tar.gz) (currently 5.4.2):
    tar zxf apache-activemq-5.4.2-bin.tar.gz
    sudo mv apache-activemq-5.4.2 /usr/local
    cd /usr/local && sudo ln -s apache-activemq-5.4.2 apache-activemq && cd apache-activemq
    sudo bin/activemq setup /etc/default/activemq && sudo chmod 600 /etc/default/activemq
    sudo adduser --system --no-create-home --shell /bin/bash activemq 
    sudo chown -R activemq:nogroup /usr/local/apache-activemq-5.4.2

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

# Configure services
* Copy Sphinx and git-daemon scripts from `gitorious/doc/templates/ubuntu` to `/etc/init.d`; change Ruby interpreter in git-daemon if you used a non-default location
    cd /var/www/gitorious/doc/templates/ubuntu
    sudo cp git-daemon git-ultrasphinx /etc/init.d
* Create poller init script (see Files below)
    sudo vi /etc/init.d/git-poller
* Copy gitorious-logrotate from `gitorious/doc/templates/ubuntu` to `/etc/logrotate.d/gitorious`
    sudo cp gitorious-logrotate /etc/logrotate.d/gitorious
* Make init scripts run at startup
    sudo ln -s /usr/local/apache-activemq/bin/activemq /etc/init.d
    sudo chmod 755 /etc/init.d/git-ultrasphinx /etc/init.d/git-daemon /etc/init.d/git-poller
    sudo update-rc.d activemq defaults
    sudo update-rc.d git-daemon defaults
    sudo update-rc.d git-ultrasphinx defaults
    sudo update-rc.d git-poller defaults
* Link the gitorious script into `/usr/local/bin`
    sudo ln -s /var/www/gitorious/script/gitorious /usr/local/bin/

# Install mod_xsendfile Apache2 module
* Compile and install
    wget 'https://tn123.org/mod_xsendfile/mod_xsendfile.c#hash(sha256:8e8c21ef39bbe86464d3831fd30cc4c51633f6e2e002204509e55fc7c8df9cf9)'
    sudo apxs2 -ci mod_xsendfile.c
* Edit `/etc/apache2/mods-available/xsendfile.load`:
    LoadModule xsendfile_module /usr/lib/apache2/modules/mod_xsendfile.so
    a2enmod xsendfile

# Configure Apache2
* Install Passenger
    sudo /opt/ruby-enterprise/bin/passenger-install-apache2-module 
* Create `/etc/apache2/mods-available/passenger.load`:
    LoadModule passenger_module /opt/ruby-enterprise-1.8.7-2010.02/lib/ruby/gems/1.8/gems/passenger-3.0.2/ext/apache2/mod_passenger.so
    PassengerRoot /opt/ruby-enterprise-1.8.7-2010.02/lib/ruby/gems/1.8/gems/passenger-3.0.2
    PassengerRuby /opt/ruby-enterprise-1.8.7-2010.02/bin/ruby
* Enable needed modules
    sudo a2enmod passenger rewrite ssl
* Restart Apache
    sudo /etc/init.d/apache2 restart
* Add a git user to MySQL:
    mysql -u root -p
        create database gitorious_production;
        grant all privileges on gitorious_production.* to 'git'@'localhost' identified by 'YOUR_PASSWORD';
* Create /etc/apache2/sites-available/gitorious (see Files below)
* Create /etc/apache2/sites-available/gitorious-ssl (see Files below)
* Disable the default sites and enable the Gitorious sites
    sudo a2dissite default default-ssl
    sudo a2ensite gitorious gitorious-ssl

# Configure Gitorious
* Run the following
    sudo adduser --system --home /var/www/gitorious --no-create-home --group --shell /bin/bash git  
    sudo chown -R git:git /var/www/gitorious          
    sudo su - git  
    mkdir .ssh  
    touch .ssh/authorized_keys  
    chmod -R go-rwx .ssh  
    mkdir tmp/pids repositories tarballs  tarballs-work
    cp config/database.sample.yml config/database.yml  
    cp config/gitorious.sample.yml config/gitorious.yml  
    cp config/broker.yml.example config/broker.yml

* Edit `config/database.yml`: Remove every section but production
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
    rake db:create
    rake db:migrate
    export PATH=/usr/local/sphinx/bin:$PATH
    rake ultrasphinx:bootstrap

    crontab -e
        PATH=/usr/local/sphinx/bin:/usr/bin:/bin
        * * * * * cd /var/www/gitorious && /opt/ruby-enterprise/bin/rake ultrasphinx:index RAILS_ENV=production

# Permissions
The gitorious documentation recommends locking down the source tree:
    sudo adduser www-data git
    cd /var/www/gitorious
    chmod -R o-rwx /var/www/gitorious
    chmod -R u+w config/environment.rb log/ tmp/ tarballs/ repositories/ db/sphinx/
    
# Finish
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
                    XSendFilePath /var/www/gitorious/tarballs
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
                XSendFilePath /var/www/gitorious/tarballs
        </IfModule>


    </VirtualHost>
    </IfModule>


/etc/init.d/git-poller
    #!/bin/sh  
    # Start/stop the git poller  
    #  
    ### BEGIN INIT INFO  
    # Provides:          git-poller  
    # Required-Start:    activemq
    # Required-Stop:  
    # Default-Start:     2 3 4 5  
    # Default-Stop:      1  
    # Short-Description: Gitorious poller  
    # Description:       Gitorious poller  
    ### END INIT INFO
     
    /bin/su - git -c "cd /var/www/gitorious;RAILS_ENV=production script/poller $@"
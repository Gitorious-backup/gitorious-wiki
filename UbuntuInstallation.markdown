Note: This documentation seems to be outdated. A more current version can be found here: [Setting up gitorious on your own server](http://cjohansen.no/en/ruby/setting_up_gitorious_on_your_own_server)

The following steps should take you from a freshly installed copy of Ubuntu Server 9.04 to a fully functioning Gitorious server.

# Install packages
    aptitude update  
    aptitude install build-essential zlib1g-dev tcl-dev libexpat-dev libcurl4-openssl-dev postfix apache2 \
        mysql-server mysql-client  apg geoip-bin libgeoip1 libgeoip-dev sqlite3 libsqlite3-dev imagemagick \
        libpcre3 libpcre3-dev zlib1g zlib1g-dev libyaml-dev libmysqlclient15-dev apache2-dev libonig-dev \
        ruby-dev rubygems  libopenssl-ruby phpmyadmin libdbd-mysql-ruby libmysql-ruby  libmagick++-dev \
        zip unzip memcached git-core git-svn git-doc git-cvs irb

# Install Ruby gems
    gem install -b --no-ri --no-rdoc rmagick chronic geoip daemons hoe echoe ruby-yadis ruby-openid \
        mime-types diff-lcs json rack ruby-hmac rake stompserver passenger
    gem install -b --no-ri --no-rdoc -v 1.3.1.1 rdiscount
    gem install -b --no-ri --no-rdoc -v 1.1 stomp

    ln -s /var/lib/gems/1.8/bin/rake /usr/bin  
    ln -s /var/lib/gems/1.8/bin/stompserver /usr/bin

# Install Sphinx
Download source from www.sphinxsearch.com (currently `http://sphinxsearch.com/downloads/sphinx-0.9.8.1.tar.gz`)
    ./configure --prefix=/usr && make all install

# Fetch Gitorious
    git clone git://gitorious.org/gitorious/mainline.git /var/www/gitorious  
-- or --  
    git clone http://git.gitorious.org/gitorious/mainline.git /var/www/gitorious

    ln -s /var/www/gitorious/script/gitorious /usr/bin

# Configure services
* Copy Sphinx and git-daemon scripts from gitorious/doc/templates/ubuntu to /etc/init.d; change Ruby interpreter in git-daemon to /usr/bin/ruby
* Create stomp init script (see Files below)
* Create poller init script (see Files below)
* Copy gitorious-logrotate from gitorious/doc/templates/ubuntu to /etc/logrotate.d/gitorious
* Make init scripts run at startup
        chmod 755 /etc/init.d/git-ultrasphinx /etc/init.d/git-daemon /etc/init.d/stomp /etc/init.d/git-poller
        update-rc.d stomp defaults
        update-rc.d git-daemon defaults
        update-rc.d git-ultrasphinx defaults
        update-rc.d git-poller defaults

# Configure Apache
* Install Passenger
        /var/lib/gems/1.8/bin/passenger-install-apache2-module 
* Create /etc/apache2/mods-available/passenger.load:
        LoadModule passenger_module /var/lib/gems/1.8/gems/passenger-2.2.5/ext/apache2/mod_passenger.so
        PassengerRoot /var/lib/gems/1.8/gems/passenger-2.2.5
        PassengerRuby /usr/bin/ruby1.8
* Enable needed modules
        a2enmod passenger  
        a2enmod rewrite
        a2enmod ssl
* Enable the SSL site
        a2ensite default-ssl
* Restart Apache
        /etc/init.d/apache2 restart
* Add a git user to MySQL: browse to https://server/phpmyadmin, to Privileges, and create a new "git" user with global create privileges. Also give it all privileges on gitorious_production.
* Create /etc/apache2/sites-available/gitorious (see Files below)
* Create /etc/apache2/sites-available/gitorious-ssl (see Files below)
* Disable the default sites and enable the Gitorious sites
        a2dissite default  
        a2dissite default-ssl  
        a2ensite gitorious  
        a2ensite gitorious-ssl

# Configure Gitorious
* Run the following

        adduser --system --home /var/www/gitorious/ --no-create-home --group --shell /bin/bash git  
        chown -R git:git /var/www/gitorious  
        
        su - git  
        mkdir .ssh  
        touch .ssh/authorized_keys  
        chmod 700 .ssh  
        chmod 600 .ssh/authorized_keys  
        mkdir tmp/pids  
        mkdir repositories  
        mkdir tarballs  
        
        cp config/database.sample.yml config/database.yml  
        cp config/gitorious.sample.yml config/gitorious.yml  
        cp config/broker.yml.example config/broker.yml  
* Edit config/database.yml: Remove every section but production
* Edit config/gitorious.yml:  Remove every section but production
  * repository path should be something like /var/www/gitorious/repositories
  * gitorious_client_port should be 80
  * gitorious_host needs to be the exact hostname that clients will use (cookies get messed up otherwise)
  * archive_cache_dir should be something like /var/www/gitorious/tarballs
  * archive_work_dir should be something like /tmp/tarballs-work
  * hide_http_clone_urls should be true (they require extra unknown setup to work)
  * is_gitorious_dot_org should be false
* Run the following
        
        export RAILS_ENV=production  
        rake db:create  
        rake db:migrate  
        rake ultrasphinx:bootstrap

        crontab -e  
            * * * * * cd /var/www/gitorious && /usr/bin/rake ultrasphinx:index RAILS_ENV=production

# Finish
As root, `/etc/init.d/apache2 restart`

# Files
/etc/apache2/sites-available/gitorious
> &lt;VirtualHost *:80&gt;  
>   ServerName your.server.com  
>   DocumentRoot /var/www/gitorious/public  
> &lt;/VirtualHost&gt;

/etc/apache2/sites-available/gitorious-ssl
> &lt;IfModule mod_ssl.c&gt;  
>   &lt;VirtualHost \_default\_:443&gt;  
>    DocumentRoot /var/www/gitorious/public  
>    SSLEngine on  
>    SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem  
>    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key  
>    BrowserMatch \"\.\*MSIE\.\*\" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0  
>   &lt;/VirtualHost&gt;  
> &lt;/IfModule&gt;

/etc/init.d/stomp
> \#!/bin/sh  
> \# Start/stop the stompserver  
> \#  
> \### BEGIN INIT INFO  
> \# Provides:          stomp  
> \# Required-Start:    $local_fs $remote_fs $network $syslog   
> \# Required-Stop:        
> \# Default-Start:     2 3 4 5  
> \# Default-Stop:      1  
> \# Short-Description: Stomp  
> \# Description:       Stomp  
> \### END INIT INFO  
>   
>   
> test -f /usr/bin/stompserver || exit 0  
>   
> . /lib/lsb/init-functions  
>   
> case "$1" in  
> start)  log_daemon_msg "Starting stompserver" "stompserver"  
>          start-stop-daemon --start --name stompserver --startas /usr/bin/stompserver --background --user git  
>          log_end_msg $?  
>         ;;  
> stop)  log_daemon_msg "Stopping stompserver" "stompserver"  
>          start-stop-daemon --stop --name stompserver  
>          log_end_msg $?  
>          ;;  
> restart) log_daemon_msg "Restarting stompserver" "stompserver"  
>         start-stop-daemon --stop --retry 5 --name stompserver  
>         start-stop-daemon --start --name stompserver --startas /usr/bin/stompserver --background --user git  
>         log_end_msg $?  
>         ;;  
> status)  
>         status_of_proc /usr/bin/stompserver stompserver && exit 0 || exit $?  
>         ;;  
> *)      log_action_msg "Usage: /etc/init.d/stomp {start|stop|restart|status}"  
>         exit 2  
>         ;;  
> esac  
> exit 0  

/etc/init.d/git-poller
> \#!/bin/sh  
> \# Start/stop the git poller  
> \#  
> \### BEGIN INIT INFO  
> \# Provides:          git-poller  
> \# Required-Start:    stomp  
> \# Required-Stop:  
> \# Default-Start:     2 3 4 5  
> \# Default-Stop:      1  
> \# Short-Description: Gitorious poller  
> \# Description:       Gitorious poller  
> \### END INIT INFO  
>   
> /bin/su - git -c "cd /var/www/gitorious;RAILS_ENV=production script/poller $@"  
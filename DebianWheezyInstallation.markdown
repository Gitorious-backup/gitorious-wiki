This article is a rip off [[DebianSqueezeInstallation]]. Many thanks to the author of that document for the work he put in. Installing and maintaining Gitorious on your own server running Debian Wheezy is much easier, as all you need (Ruby 1.8, Sphinx 2.x, ActiveMQ) can be installed and updated via Debian's package manager.
# Install packages
    # If you use MariaDB instead of MySQL, s/mysql/mariadb/g
    apt-get install build-essential ruby1.8-dev ruby-switch libapache2-mod-passenger libapache2-mod-xsendfile activemq mysql-server mysql-client sphinxsearch git apg imagemagick unzip libmysqlclient-dev libxslt1-dev

# Fetch Gitorious

* Clone repository:
        git clone git://gitorious.org/gitorious/mainline.git /var/www/gitorious
        cd /var/www/gitorious
* With release 2.0.0 gitorious began [tagging releases](https://gitorious.org/gitorious/pages/Versioning). So if you feel the need for a more stable version of Gitorious checkout the latest tag.
        git reset --hard `git tag | tail -n 1`
* Fetch submodules:
        git submodule update --init

# Switch ruby version
Gitorious is using Rails 2.3.5 and therefore is only compatible with Ruby 1.8.

    ruby-switch --set ruby1.8

# Install Ruby gems
    bundle install

# Create system user
* Add `git` system user:
        adduser --system --home /var/www/gitorious --no-create-home --group --shell /bin/bash git
* Assign ownership of  /var/www/gitorious and its contents to `git`:
        chown -R git:git /var/www/gitorious
* Create missing files and directories:
        su - git
        mkdir .ssh tmp/pids repositories tarballs  tarballs-work
        touch .ssh/authorized_keys
        chmod -R go-rwx .ssh

# Configure Gitorious
* Create config files (as user `git`):
        cp ~/config/database.sample.yml ~/config/database.yml
        cp ~/config/gitorious.sample.yml ~/config/gitorious.yml
        cp ~/config/broker.yml.example ~/config/broker.yml
* Edit `config/database.yml`: Remove every section but production
  * `username` must be `gitorious`
  * `password` must be a new password. You can call `apg -m 16` to generate it.
* Edit `config/gitorious.yml`:  Remove every section but production
  * `repository_base_path` must be something like `/var/www/gitorious/repositories`
  * `cookie_secret` must be a long random unique string. You can call `apg -m 64` to generate it.
  * `gitorious_client_port` must be 80
  * `gitorious_host` needs to be the exact hostname that clients will use (cookies get messed up otherwise)
  * `archive_cache_dir` must be `/var/www/gitorious/tarballs`
  * `archive_work_dir` must be `/var/www/gitorious/tarballs-work`
  * `hide_http_clone_urls` should be `true` (they require extra unknown setup to work)
  * `is_gitorious_dot_org` must be `false`

Also see [[Configuration]].

# Create database and sphinx index
* Create MySQL database and user:
        mysql -u root -p
            CREATE DATABASE gitorious;
            GRANT ALL PRIVILEGES ON gitorious.* to 'gitorious'@'localhost' identified by 'YOUR_PASSWORD';
* Bootstrap database and sphinx index (as user `git`):
        export RAILS_ENV=production
        bundle exec rake db:create
        bundle exec rake db:migrate
        bundle exec rake ultrasphinx:bootstrap
* Add cronjob for updating the sphinx index (as user `git`):
        crontab -e
            * * * * * cd /var/www/gitorious && bundle exec rake ultrasphinx:index RAILS_ENV=production

If rake aborts with `uninitialized constant ActiveSupport::Dependencies::Mutex`, [read here](https://groups.google.com/forum/?fromgroups#!topic/gitorious/uzPUxzfZIJQ).

# Install scripts and start services
* Create symlinks for the git-daemon and git-ultrasphinx init scripts:
        ln -s /var/www/gitorious/doc/templates/ubuntu/git-daemon /etc/init.d/
        ln -s /var/www/gitorious/doc/templates/ubuntu/git-ultrasphinx /etc/init.d/
* Create poller init script (see Files below):
        vi /etc/init.d/git-poller
        chmod 755 /etc/init.d/git-poller
* Create symlink for the logrotate script:
        ln -s /var/www/gitorious/doc/templates/ubuntu/gitorious-logrotate /etc/logrotate.d/gitorious
* Create symlink for the gitorious script into `/usr/local/bin`:
        ln -s /var/www/gitorious/script/gitorious /usr/local/bin/
* Start services:
        service git-daemon start
        service git-ultrasphinx start
        service git-poller start
* Make init scripts run on startup:
        insserv git-daemon git-ultrasphinx git-poller

# Configure ActiveMQ
* Create new ActiveMQ instance:
        cd /etc/activemq/instances-available
        cp -r main gitorious
* Edit `/etc/activemq/instances-available/gitorious/activemq.xml`. Remove the predefined transportConnector and add a new one using the stomp protocol:
        <transportConnectors>
            <transportConnector name="stomp" uri="stomp://localhost:61613"/>
        </transportConnectors>
* Enable ActiveMQ instance:
        cd /etc/activemq/instances-enabled
        ln -s ../instances-available/gitorious .
* Restart ActiveMQ:
        service activemq restart

# Configure Apache2
* Create /etc/apache2/sites-available/gitorious (see Files below)
* Create /etc/apache2/sites-available/gitorious-ssl (see Files below)
* Enable the Gitorious sites:
        a2ensite gitorious gitorious-ssl
* Restart Apache:
        service apache2 restart

# Console User Administration:
     su - git
     env RAILS_ENV=production script/console

* Delete a User
        > user = User.find_by_login "Username"
        > user.destroy
* Create Roles
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
    # Required-Start:    activemq mysql
    # Required-Stop:  activemq mysql
    # Default-Start:     2 3 4 5
    # Default-Stop:      1
    # Short-Description: Gitorious poller
    # Description:       Gitorious poller
    ### END INIT INFO
     
    /bin/su - git -c "cd /var/www/gitorious;RAILS_ENV=production script/poller $@"
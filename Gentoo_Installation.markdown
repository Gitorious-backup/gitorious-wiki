First install git.

<pre>

emerge -va git

</pre>

Clone the portage overlay that has the Gitorious ebuild.

<pre>

git clone git://github.com/adrenlinerush/portage_overlay.git

</pre>

Edit /etc/make.conf and add the following line.

<pre>

PORTDIR_OVERLAY="/path/to/clone"

</pre>

Install, configure, start, and tell to start on boot mysql.  If you don't give it a root password my ebuild will create the database, gitorious user and run rake.

<pre>

emerge -va dev-db/mysql
emerge --config =dev-db/mysql-5.0.84-r1
rc-update add mysql default
/etc/init.d/mysql start

</pre>

Edit /etc/portage/package.keywords to unmask gem dependancies.

<pre>

=dev-ruby/diff-lcs-1.1.2 ~amd64
=dev-ruby/mime-types-1.16 ~amd64
=dev-ruby/oniguruma-1.1.0 ~amd64
=dev-ruby/ruby-openid-2.1.7 ~amd64
=dev-ruby/ruby-yadis-0.3.4 ~amd64
=dev-ruby/ruby-hmac-0.3.2 ~amd64
=dev-ruby/stompserver-0.9.9 ~amd64
=dev-ruby/eventmachine-0.12.10 ~amd64
=dev-ruby/mime-types-1.16-r1 ~amd64
=dev-ruby/net-ssh-2.0.16 ~amd64
=dev-ruby/hoe-2.4.0 ~amd64
=dev-ruby/json-1.2.0 ~amd64
=dev-ruby/Ruby-MemCache-0.0.4 ~amd64
=dev-ruby/IO-Reactor-0.0.6 ~amd64
=dev-ruby/rmagick-2.12.2 ~amd64
=dev-ruby/rack-1.0.1 ~amd64
=dev-ruby/mysql-ruby-2.8 ~amd64

</pre>

Edit /etc/portage/package.use to set the use flags for the custom ebuilds.

<pre>

www-misc/passenger nginx
www-servers/nginx gitorious ssl passenger
www-apps/gitorious mysql

</pre>

Now install Gitorious.

<pre>

emerge -va gitorious

</pre>

Tell required services to start on boot.

<pre>

rc-update add stompserver default
rc-update add nginx default
rc-update add memcached default

</pre>

Now add a FQDN to the hosts file.

<pre>

nano /etc/hosts
127.0.0.1 	localhost git.localhost

</pre>

You can use something other than git.localhost but you will also have to edit the /var/www/gitorious/site/config/gitorious.yml file and the /etc/nginx/nginx.conf file if you do.  Also alternatively if you have dns you could add it there instead of the hosts file.

Then you can either reboot or manually start the items in the git users crontab.

<pre>

shutdown -r now

</pre>

After reboot you should be able to open a browser and goto http://git.localhost or whatever you named the site.




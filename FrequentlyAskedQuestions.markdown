# Frequently asked questions for your own Gitorious installation

## Login isn't working

Since Gitorious supports several sites we use a wildcard hostname when sending cookies, including for sessions. This means that in order for users to log in they need to use a valid host/domain name, not the IP address or (in some cases/browsers) special addresses like "localhost".

The preferred way to solve this is to enter a fully qualified domain name under gitorious_host in config/gitorious.yml **and** use this in your web browser. Given this entry in config/gitorious.yml:

    gitorious_host: git.bigcorp.com

you should use the address http://git.bigcorp.com/ when browsing the site.

If you for some reason cannot use a fully qualified hostname for your server, you can modify config/initializers/session_store.rb and change domain to the hostname or IP your server uses.
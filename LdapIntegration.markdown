LDAP integration
================

Gitorious gained support for LDAP authentication in version 2.1.
There are probably as many LDAP patterns being used as there are LDAP
servers, so getting it to work with your infrastructure will need a
little configuration.

The LDAP support in Gitorious is known to work with typical Active
Directory.

Overview
--------

This integration lets your users authenticate using their LDAP
credentials. Non-registered users will be automatically registered on
your Gitorious servers upon first successful authentication. User
passwords are not stored in Gitorious.

When users are auto-registered, Gitorious will attempt to look up user
information from LDAP - email, full name etc.

Configuration
-------------

Configuration of custom authentication mechanisms in Gitorious is done
by editing `config/authentication.yml` - there's a sample file
included which explains the various settings defined there. This file
contains a section for each Rails environment, like database.yml.

The first setting you need to decide whether to use the built-in
(database-backed) authentication alongside LDAP. If you choose not to,
any user logging in will need an account on your LDAP server. To
disable built-in authentication, add

    disable_default: true

Next, there is a list of "custom" authentication methods, of which
LDAP is currently the only supported one. Look at the configuration
file for explanations on the various settings. 

Testing your setup
------------------

Once you have configured `authentication.yml` you will want to try
your settings - for this there is a script in
`script/test_ldap_connection` which will try to connect to your LDAP
server, run it like so:

    bundle exec script/test_ldap_connection USERNAME PASSWORD
    
You will need at least to configure the following in `authentication.yml`:

* server
* base_dn

If you're having problems using this or have ideas for improvements,
send a message to the mailing list!


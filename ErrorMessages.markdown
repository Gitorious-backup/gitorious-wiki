If you have a problem in your local Gitorious installation, the first thing to do is check the log files.
Gitorious and its tools have several log files:

* `<GITORIOUS_ROOT>/log/<RAILS_ENV>.log` (gitorious itself)
* `<GITORIOUS_ROOT>/db/sphinx/log/* (ultrasphinx)
* `<GITORIOUS_ROOT>/tmp/pid/* (i.e. git-poller)

## Known error messages

Please note that this list is not complete; other parts of the framework may issue other warnings. 

### Invalid subdomain name <subdomain>. Session cookies will not work!

You have specified an illegal `gitorious_host` in your `config/gitorious.yml` file. For sessions to work, you need to supply a valid domain name for this key. Valid values contain at least one dot (.). 

If your Gitorious server is located as gitorious.example.com, you should enter `gitorious.example.com`, not `gitorious`, otherwise Gitorious will issue a warning on startup.

### The specified gitorious_host is reserved in Gitorious

Gitorious uses a special hostname for cloning repositories over HTTP (currently `git`). If you use this value as `gitorious_host` in `config/gitorious.yml`, HTTP cloning will not work and Gitorious will issue a warning on startup. 

Either use another name or change the value of `HTTP_CLONING_SUBDOMAIN` in `config/gitorious.yml`. 
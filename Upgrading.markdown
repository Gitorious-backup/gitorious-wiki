# Upgrading gitorious

If you want to upgrade gitorious to a version newer than 92bb70a1, then these instructions are for you!

First things first, backup your database.

### New library/server depencies

  * An [ActiveMessaging](http://code.google.com/p/activemessaging/wiki/ActiveMessaging) compatible queue server. Gitorious.org runs [ActiveMQ](http://activemq.apache.org/) and STOMP, [stompserver](http://stompserver.rubyforge.org/) might be a nice flyweight alternative for your needs
  
  
### Gems

you should have at least the following gems, some of these may vary depending on what database and deployment server you chose (mysql vs. postgres, passenger vs. mongrel etc)

    chronic (0.2.3)
    daemons (1.0.10)
    diff-lcs (1.1.2)
    echoe (3.1.1)
    fastthread (1.0.7)
    geoip (0.8.0)
    highline (1.5.0)
    hoe (1.12.2)
    mime-types (1.16)
    mysql (2.8.1):
    oauth (0.3.4)
    passenger (2.2.2)
    rack (1.0.0)
    rake (0.8.5)
    rdiscount (1.3.4, 1.3.1.1)
    RedCloth (4.1.9)
    rmagick (2.9.1)
    ruby-hmac (0.3.2)
    ruby-openid (2.1.6)
    ruby-yadis (0.3.4)
    rubyforge (1.0.3)
    rubygems-update (1.3.3)
    sqlite3-ruby (1.2.4)
    stomp (1.1)

### Migrating your data

Once you've checkout the new Gitorious codebase, start up the console:

(if you're running running you'd want to run `ruby1.9 script/console production --irb=irb1.9`)

    >> rs = Repository.all.select{|r| r.project.nil? }.each &:delete
    >> SshKey.all.select{|s| s.user.nil?}.each &:delete

This is needed because gitorious previously missed some `:dependent` hooks on the Project and User classes, if you don't do this, migrations will fail.

### Changes in Gitorious.yml

The config/gitorious.yml file now is organized like config/database.yml; that is a section for each of your Rails environments. This will typically lead to an error like 

     undefined method `[]' for nil:NilClass

To resolve this, add a section for at least the development env, like so:

    development:
        gitorious_client_port: 3000
        gitorious_client_host: gitorious.local

Check config/gitorious.sample.yml for a sample.

### rake migrate


    $ env RAILS_ENV=production rake db:migrate 


### Put users in the activated state

If you're running a private install, you probably want to put all your users in the `terms_accepted` state right away, in `script/console`:

    >> User.update_all("aasm_state = 'terms_accepted'")


### Migrate repository hashed_path

For performance reasons when there's lots of repositories (for instance on gitorious.org), the on-disk repository locations are now a sharded hash. While there is a `script/shard_git_repositories_by_hash` you can just update the hashed_path attribute of all the repositories to point at their existing location, in `script/console` paste the following:

    Repository.find_each do |repo|
      next unless repo.project
      repo.hashed_path = File.join(repo.project.slug, repo.name)
      puts "##{repo.id} => " + repo.hashed_path
      repo.save!
    end


### Start the queue poller/worker

The poller script is what fetches jobs from the queue server and processes them, it invokes a processor from `app/processors/` and it logs into log/message_proccessing.log

    $ script/poller start


### Start the git-daemon (optional)

Since the on-disk repository locations might be different from the url request path, the git-daemon in `script/git-daemon` understands this and should be used if you wish to provide anonymous read-only access to your repositories.
If you don't wish for that (and for an internal install you probably wouldn't), simply don't start the git-daemon.


### That's it

Congrats, hopefully your existing gitorious installation should be up and running, but if you run into any issues drop an email on the mailing list.

# Upgrading old versions of Gitorious

If your version of Gitorious is older than February 3rd 2009 (92bb70a1), then these instructions are for you!

First things first, backup your database.

### New library/server depencies

  * An [ActiveMessaging](http://code.google.com/p/activemessaging/wiki/ActiveMessaging) compatible queue server. Gitorious.org runs [ActiveMQ](http://activemq.apache.org/) and STOMP, [stompserver](http://stompserver.rubyforge.org/) might be a nice flyweight alternative for your needs

### Update the Gitorious source

    >> cd /path/to/gitorious
    >> git pull git://gitorious.org/gitorious/mainline.git master

### Gems

Gems are handled by [Bundler](http://gembundler.com/). For the least number of surprises, please remove any system gems already installed (beware, this may cause problems if you have other apps relying on the same set of gems). The install bundler, and bundle Gitorious' dependencies:

    gem install bundler
    bundle install

### Migrating your data

Once you've checked out the new Gitorious codebase, start up the console:

(if you're running Ruby 1.9 from a non-standard location you'd want to run `ruby1.9 script/console production --irb=irb1.9` or something similar)

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

Congrats, hopefully your existing gitorious installation should be up and running, but if you run into any issues drop an email on [the mailing list](http://groups.google.com/group/gitorious).

To keep your Gitorious up to date, refer to the new [[Upgrading]] guide.
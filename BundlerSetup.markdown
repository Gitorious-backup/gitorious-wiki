# Using Bundler with Gitorious

Since January 24th, Gitorious manages gems with [Bundler](http://gembundler.com/). This means that if you update to a version newer than this commit without preparing your environment, your install will crash.

If you don't have other applications depending on your global gems, we recommend you remove them, or use [RVM gemsets](http://beginrescueend.com/gemsets/basics/) to create a sandbox for Gitorious. This will cause less frustration in the long run.

Then install the Bundler gem:

    >> gem install bundler

And bundle things up:

    >> cd /path/to/gitorious
    >> bundle install

It is recommended that every binary that executes Gitorious Ruby code is executed with `bundle exec`, especially if you have other gems installed on the system. An example of starting the server:

    >> `bundle exec script/server`

This ensures (well, mostly) that system wide gems won't interfere with your app, and that it loads with a usable environment.
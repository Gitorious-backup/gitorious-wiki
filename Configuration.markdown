
## Time zone / Local time
Times for i.e. commits are stored as UTC time in the database. By default, Gitorious displays them as UTC times on the web site.
This setting can be changed so times are local to the server or userbase:

1. List all local time zones: `rake time:zones:local`
2. Edit `config/environment.rb`
3. Set `config.time_zone` to the local time zone, i.e. "Berlin"
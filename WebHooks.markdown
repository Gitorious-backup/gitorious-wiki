# Web hooks in Gitorious 

Note: This page is currently intended for local Gitorious installations, since this feature is not offered as part of Gitorious.org yet.

Gitorious requires several [Git hooks](http://www.kernel.org/pub/software/scm/git/docs/githooks.html) to work properly; these are used for checking permissions, logging push events and so on. Users are free to other hooks to their local repositories, but sharing these can be cumbersome. 

Many users, however, have asked how to integrate Gitorious with bug trackers, microblog updates etc, and the web hooks feature in Gitorious could be a solution for this.

Note that you need to set up a web server that receives and handles this information; since the data is in JSON format, most languages and programming environments should make this quite easy.


The overall functionality is as follows:

* each repository can have one or several web hooks set up
* whenever a commit is pushed to Gitorious, Gitorious will attempt to connect to the URL associated with the web hooks - if any 
* Gitorious will send a HTTP payload in JSON format, containing a summary of who pushed

## An example of the payload

Whenever a user pushes one or several commits to Gitorious, one JSON request will be made. This is a HTTP POST request with a single parameter: `payload` containing the JSON data about the push. This is an example of such a payload for a fictional repository (Login, edit this page, and copy the payload from the text box to get valid JSON):

<code>
{
  "after": "df5744f7bc8663b39717f87742dc94f52ccbf4dd", 
  "before": "b4ca2d38e756695133cbd0e03d078804e1dc6610", 
  "commits": [
    {
      "author": {
        "email": "jason@nospam.org", 
        "name": "jason"
      }, 
      "committed_at": "2012-01-10T11:02:27-07:00", 
      "id": "df5744f7bc8663b39717f87742dc94f52ccbf4dd", 
      "message": "added a place to put the docstring for Book", 
      "timestamp": "2012-01-10T11:02:27-07:00", 
      "url": "http:\/\/gitorious.org\/q\/mainline\/commit\/df5744f7bc8663b39717f87742dc94f52ccbf4dd"
    }
  ], 
  "project": {
    "description": "a webapp to organize your ebook collectsion.", 
    "name": "q"
  }, 
  "pushed_at": "2012-01-10T11:09:25-07:00", 
  "pushed_by": "jason", 
  "ref": "new_look", 
  "repository": {
    "clones": 4, 
    "description": "", 
    "name": "mainline", 
    "owner": {
      "name": "jason"
    }, 
    "url": "http:\/\/gitorious.org\/q\/mainline"
  }
}
</code>
 
##Testing web hooks


To see the actual data sent from your Gitorious server, [Postbin](http://postbin.org/) provides a free and easy solution, simply go to their site and click "Make a Postbin". You will immediately be given a URL you can push data to, and all data is displayed by visiting this URL in your browser. 


## Enabling web hooks for your local Gitorious installation

Since this feature needs some testing and feedback, Gitorious does not currently offer a UI for adding/maintaining web hooks. 

Adding one is done from the Rails console (can be started from the root of the Gitorious installation, example: `RAILS_ENV=production bundle exec script/console`). Changes made in the console are saved to the underlying database and are persistent between Gitorious startups.

To set up the hook once you're in the console run:
<code>
    project = Project.find_by_slug "gitorious"
    repository = project.repositories.find_by_name "mainline"
    hook = repository.hooks.build
    hook.user = repository.user
    hook.url = "http://www.postbin.org/wqpx3l"
    hook.save
</code>

Whenever you push some commits to this repository, a request will be made to the server you specified. The Hook object in the database keeps track of the number of successful and failed requests, but URLs that continue to fail will not be deactivated.

**Note!** Postbin service has been renamed to [RequestBin](http://requestb.in).
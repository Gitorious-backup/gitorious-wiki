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

Whenever a user pushes one or several commits to Gitorious, one JSON request will be made. This is a HTTP POST request with a single parameter: `payload` containing the JSON data about the push. This is an example of such a payload for a fictional repository:

<code>
{
  "after": "34f5a766e48bd270037350961d79fa59f9b746d7", 
  "before": "d46f9c3ee6d4eca4e8248f4cabf782e1f1e79fa9", 
  "commits": [
    {
      "author": {
        "email": "marius@shortcut.no", 
        "name": "Marius Mathiesen"
      }, 
      "committed_at": "2010-08-10T09:06:08Z", 
      "id": "34f5a766e48bd270037350961d79fa59f9b746d7", 
      "message": "Must be", 
      "timestamp": "2010-08-10T09:06:08Z", 
      "url": "http:\/\/gitorious.here\/web-hooks\/web-hooks\/commit\/34f5a766e48bd270037350961d79fa59f9b746d7"
    }, 
    {
      "author": {
        "email": "marius@shortcut.no", 
        "name": "Marius Mathiesen"
      }, 
      "committed_at": "2010-08-10T09:05:56Z", 
      "id": "0ceada81a4300abdf703724ea045d15055943c6b", 
      "message": "Are we having fun yet?", 
      "timestamp": "2010-08-10T09:05:56Z", 
      "url": "http:\/\/gitorious.here\/web-hooks\/web-hooks\/commit\/0ceada81a4300abdf703724ea045d15055943c6b"
    }
  ], 
  "project": {
    "description": "Trying out web hooks", 
    "name": "web-hooks"
  }, 
  "pushed_at": "2010-08-10T09:06:13Z", 
  "pushed_by": "johan", 
  "ref": "refs\/heads\/master", 
  "repository": {
    "clones": 0, 
    "description": "A simple repository", 
    "name": "web-hooks", 
    "owner": {
      "name": "johan"
    }, 
    "url": "http:\/\/gitorious.here\/web-hooks\/web-hooks"
  }
}
</code>
 
##Testing web hooks


To see the actual data sent from your Gitorious server, [Postbin](http://postbin.org/) provides a free and easy solution, simply go to their site and click "Make a Postbin". You will immediately be given a URL you can push data to, and all data is displayed by visiting this URL in your browser. 


## Enabling web hooks for your local Gitorious installation


Since this feature needs some testing and feedback, Gitorious does not currently offer a UI for adding/maintaining web hooks. Adding one is done from the console, like this:

  project = Project.find_by_slug "gitorious"
  repository = project.repositories.find_by_name "mainline"
  hook = repository.hooks.build
  hook.user = repository.user
  hook.url = "http://www.postbin.org/wqpx3l"
  hook.save

Whenever you push some commits to this repository, a request will be made to the server you specified. The Hook object in the database keeps track of the number of successful and failed requests, but URLs that continue to fail will not be deactivated.
Please feel free to add bugs and/or feature requests below.

## Bugs
* diff display sometimes hilights wrong parts of lines (or no parts at all) when tab is changed to spaces in the middle of lines. See changes to track_info.h here: http://gitorious.org/jasonwoof/cmus/commit/ef82f70f073e537e3677ad0d79d50acb4c667d87
* Edit team page lacks button to "Delete current team image" (the edit person page has this) [DONE]
* tarballs should contain one directory (preferably matching the basename of the tarball) with all the files in it
* display the year of commits [[merged](http://gitorious.org/gitorious/mainline/commit/30f136bb75230b2d607bbe64581e60624a5e9269)] [DONE!]
* When showing commits for a branch, whatever occurs after the "-" in the branch name is ignored and the commits for /^(.*?)-/ are shown
  * Example: http://gitorious.org/~br3nda/laconica/br3nda/commits/0.8.x-attachmentfix -> shows the commits for 0.8.x (note that if the branch 0.8.x was deleted, it would show the commits for 0.8.x-attachmentfix)
* There's a bug in the Markdown parser/formatter. Sometimes the code block is accidentally closed. Here's a minimal example:
  * foo
        bar
    }
* "Activities" lists display only the first line of a commit message, whether that commit message contains text or not.   To fix:  remove leading whitespace 
  from commit messages before displaying in activities list.
* On an 800x480 screen resolution (i.e. an Eee), the login form is completely pushed off the right side of the page - and is still hidden when scrolling to the right (I have to visit /sessions manually to be able to log in when I'm using my Eee)
* Branches with the '+' character don't show up in the web interface.
* Branches with the '=' character don't show up in the dropdown when putting in a merge request
* Merge request numbering got messed up around end of nov 2009. E.g. http://qt.gitorious.org/qt/qt/merge_requests/402 shows "2374" as the MR number in the top navigation bar, and indeed previous merge requests had a higher number, somehow it went back to small numbers again. I guess this one -should- have been 2374, not 402. [DONE]
* When gitorious is deployed locally, under non-standard port (example: http://git.example.com:3000/) clicking on "Your Dashboard" link removes the reference to the port (Example: it becomes "http://git.example.com/") and user ends up on a "page not available" page. The issue becomes worse (more confusing for users) when gitorious is hosted from behind a reverse proxy (for example from behind NGINX's reverse proxy) where :80 is actually a site-wide home page or is some other intranet application (SugarCRM in our case). There is a bypass for the issue. Instead of clicking on "Your Dashboard" user must click on Gitorious banner, which takes the user to the same place as the "Your Dashboard" would, but preserves the port as part of the url. (Added Jan 23, 2010)
* Gitorious doesn't work on PostgreSQL, see [this bug report](http://gitorious.lighthouseapp.com/projects/53751/tickets/2-group-by-query-error-on-postgresql).
* if you have a git project with no master branch, or name your master something else, then go to view the project source tree the browser gets stuck in an endless loop:  "The webpage at http://gitorious.org/ubuntuone/hacking/trees/HEAD has resulted in too many redirects. Clearing your cookies for this site or allowing third-party cookies may fix the problem. If not, it is possibly a server configuration issue and not a problem with your computer."

## Feature Requests
* Allow users to set their time zone and display all times accordingly.
* ability to have a mix of public and private projects
* on project/repo pages where it says the license, it should link to the license page. The [FSF recently added RDF data for license permissions to the html versions of their licenses](http://www.fsf.org/blogs/licensing/2009-06-rdf) so we should get that rel="license" in there for the web crawlers.
* commits on main project page and main repo page (eg [here](http://gitorious.org/wfpl) and [here](http://gitorious.org/wfpl/wfpl)) should recognize e-mail aliases of committers (as set under "manage aliases"). Specifically, these should link their name to their gitorious account and show the [gr]avitar from the primary e-mail
* make diff's "side-by-side" mode show changed lines with one table row. eg [here](http://gitorious.org/wfpl/wfpl/commit/532d75b922c5e4e2e39172b6fe3c4b2153ffc28d?diffmode=sidebyside).
* set up an Admin profile - superuser installed by default [DONE!]
* set up an Admin interface - manage users, daemon status [DONE!]
* make it more configurable (logo, basic color theme at least)
* make it private (enforce login instead of being always open) [DONE!]
    * maybe not mix public and private but an option to choose between them
* i18n/l10n [DONE]
* improve the CentOS and Ubuntu init scripts file
* create unattended CentOS and Ubuntu scripts to make installation even easier
   * maybe deprec or capistrano is the way to go here? deprec atleast have alot of pre-made receipts for deploying rails apps
* Watcher's List/Favorite projects [DONE]
* Feature to Delete (actually Hide) a repository
* async task to compile lists/rss
    * newest projects
    * projects with more activity
    * forking history visualization (graphical)
* a simple bugtracker implementation inside gitorious
* Integration with a bugtracker (or ticketing system)
* Some sort of 'powered by gitorious' banners. I would be more than glad of placing it on my project's website 
    * feel free to use http://gitorious.org/images/powered_by_gitorious.png (licensed under the same as the logo, see the about page)
* It would be nice if repositories in a project were ordered (eg http://gitorious.org/mer )
* Possibility to add images (e.g. for screen shots) to Wiki pages
* New symbolic-ref 'Upstream' to (optionally) add to a repository a ref that would point to the branch that all downstream merge requests should target.  This symbolic ref could be changed via the 'Edit the repository' page just like the HEAD ref can be changed now.  The difference is that when this new ref changes, all merge requests that are currently open on the existing 'Upstream' would be automatically closed with a state change and a notification that the merge request(s) will need to be changed to target the new 'Upstream' branch instead.  The purpose is to have a workflow where the 'Upstream' itself might need to be rebased from time to time against something even more upstream and to simultaneously deny force pushing.
* Stick links to my projects and repos on the homepage - 90% of the time I come to Gitorious to see the projects I'm involved with, not to browse the most popular or recently-pushed-to projects
* Projects should have the option to set a project image
* Site admin should have access to all project properties not only via database backend, but also via gitorious frontend - this could be accomplished with an impersonate user option, realized like e.g. in bugzilla.
* API (perhaps compatibility with/similarity to [GitHub API](http://develop.github.com/) (link to version 2))
* I am getting swamped by merge request-emails of a project I'm just partly involved with and therefor it would be nice to have a per-group option to receive emails from clone/merge-requests. [DONE]
* Cloned projects should do a nightly fetch-pack.  Imagine me creating a clone on the site, I download it and weeks later want to merge with upstream. I need to download the diffs from the original project. Thats fine. Then I merge locally and to my surprise I need to upload all those objects again (slow!).  What about a nightly fetch pack so the server already has those objects.
* Code editing in gitorious source view would be a cool feature for small (typo) fixes.
* Gitorious' MQ system should support rabbitmq. RabbitMQ's stomp plugin works ok with its bundled test cases, and I don't see any obvious error or reason why it wouldn't work
* If a more comprehensive API is not possible, at least allow Merge Request status changes via signed e-mail from reviewers approved for a repository.
* It would be nice to get the git-http-backend implemented into gitorious, so all repos added to it could be fetched, pulled, pushed, etc over the http or https protocol.
* README "à la github" where you can create a README.md in a wiki markup and used as a source for the webpage at gitorious. That would allow users from github to share more easily with gitorious.org. http://github.com/guides/readme-formatting

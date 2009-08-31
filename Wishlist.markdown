Please feel free to add bugs and/or feature requests below.

## Bugs
* Edit team page lacks button to "Delete current team image" (the edit person page has this) [DONE]
* tarballs should contain one directory (preferably matching the basename of the tarball) with all the files in it
* display the year of commits [[merged](http://gitorious.org/gitorious/mainline/commit/30f136bb75230b2d607bbe64581e60624a5e9269)] [DONE!]
* When showing commits for a branch, whatever occurs after the "-" in the branch name is ignored and the commits for /^(.*?)-/ are shown
  * Example: http://gitorious.org/~br3nda/laconica/br3nda/commits/0.8.x-attachmentfix -> shows the commits for 0.8.x (note that if the branch 0.8.x was deleted, it would show the commits for 0.8.x-attachmentfix)

## Feature Requests
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
* Watcher's List/Favorite projects
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
* API
* I am getting swamped by merge request-emails of a project I'm just partly involved with and therefor it would be nice to have a per-group option to receive emails from clone/merge-requests.
Please feel free to add bugs and/or feature requests below.

## Bugs
* Edit team page lacks button to "Delete current team image" (the edit person page has this) [DONE]
* tarballs should contain one directory (preferably matching the basename of the tarball) with all the files in it
* display the year of commits [[merged](http://gitorious.org/gitorious/mainline/commit/30f136bb75230b2d607bbe64581e60624a5e9269)] [DONE!]

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
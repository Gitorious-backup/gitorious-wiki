Some of Gitorious' information can be extracted programmatically.

# Project list
URL: [/projects.xml](https://gitorious.org/projects.xml)

Contains a partial list of projects, their repositories and clone URLs.
Accepts a page parameter, e.g. [`/projects.xml?page=2`](https://gitorious.org/projects.xml?page=2) - but there is no way to get the number of pages.

# Project details
URL: [/projectname.xml](https://gitorious.org/gitorious.xml)

Contains the details to the project: Bugtracker, Mailinglists, Repositories


# Repository details
URL: [/projectname/repositoryname.xml](https://gitorious.org/gitorious/mainline.xml)

Contains creation and last push date, owner, clone and push urls.
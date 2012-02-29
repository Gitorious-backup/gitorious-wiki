# Private repositories #

As of version 2.2.0, Gitorious supports private repositories. This is a feature where users can control who gets access to their projects and repositories. The feature must be enabled with a setting in the `config/gitorious.yml` configuration file, and is not enabled by default. This feature will also not be available on gitorious.org.

## Installing ##

Private repositories will ship with Gitorious 2.2.0. Until then, you can run off of the `private-repos` branch to test it out:

```
git checkout -b private-repos
git pull git://gitorious.org/gitorious/mainline.git refs/heads/private-repos
bundle
bundle exec rake db:migrate
bundle exec rake assets:clear
touch tmp/restart.txt
```

This assumes you are using [Passenger](http://www.modrails.com/). Adjust accordingly if you are not.

## Enabling ##

To enable private repositories, open your `config/gitorious.yml` and add the following setting to the desired environment:

```
  enable_private_repositories: true
```

Then restart your server.

## How does it work? ##

When private repositories are enabled, Gitorious will authorize most content through the new `Gitorious::Authorization` module. This module is designed to take pluggable authorization strategies. Currently, the only implementation is database backed authorization (in [database_authorization.rb](https://gitorious.org/gitorious/mainline/blobs/private-repos/lib/gitorious/authorization/database_authorization.rb)).

### Private projects ###
When creating a new project, you will see a new checkbox that you can check to make the project private. When the project is private, only its owner can browse it, see events from it, clone and browse repositories under it and so on. After creating the project, you can use the "Manage access" button in the right column to add collaborators (users and groups) that will be able to browse the project as well.

A project can be made private retroactively also. Click the "manage access" button in the right column on the project page and then hit "make private". Making the project will make every repository under it private as well - including repository clones. This means that making a project private may block users out of their repository clones if they are not given access to the project as well. Use with care!

If you want to open a private project to the public, simply go to "manage access", then hit "make public". Note that "public" in this case means public within Gitorious. If your Gitorious is running in "private mode", only registered and logged in users will actually be able to see "public" projects.

### Private repositories ###

Private repositories work in much the same way as projects, only on the repository level (duh!) You can make a repository private right from the get-go by ticking the checkbox in the new repository form. To retroactively make a repository private, go to the repository page, click "manage read access" and then "make private".

You can add collaborators - both users and groups - to your private repositories, and they will not only be able to browse the repository on Gitorious, but also clone the repository over ssh/git/http (depending on your specific configuration).

Note that adding a member to your repository does not automatically give them commit rights - the old committerships model is still used for this. To add a committer (or reviewer/admin) to a private repository, first grant them read access (click the "manage read access" button on the repo page), then grant them committerships.

Unlike with projects (as explained above), retroactively making a repository private can not block users from their clones. You should be aware that making a repository private after people have cloned it will not entirely hide the repository's existence. The clones will have references, and even links, to the original repository. However, these links will always take unauthorized users to a "unauthorized" page. Additonally, no news items or other data will ever leak from a private repository (if they do, [report a bug](http://issues.gitorious.org)).

When you clone a private repository (that you have been given read access to, obviously), your repository clone will be private, and will include all the members of the original repository as members by default. You can of course remove or add members as you like.

## Git over HTTP ##

When using private repositories, we recommend disabling HTTP access. The current Git over HTTP implementation is slightly limited, and only supports database authorization directly. This means that it works, but if you plan to control access using e.g. LDAP, you must disable Git over HTTP.

Eventually, Gitorious will ship with Git over HTTP using [mutt](https://gitorious.org/gitorious/mutt), which can support sophisticated authorization, which will make the full feature set of private repositories available over authenticated HTTP as well.

## The git:// protocol ##

`git://` cloning may still be used with private repositories, but because it is an anonymous protocol, it can only be used to pull public repositories. The `git-daemon` has full authorization support, which means it will prevet access to repositories that are locked down through other measures than database authorization.

## Git over SSH ##

Git over SSH works as expected.

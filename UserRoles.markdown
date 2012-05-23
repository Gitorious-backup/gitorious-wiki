# About Roles
Users in Gitorious may have several roles:

## Global Roles
* **Administrator**
  * Has access to the user administration page
  * Can _not_ change projects or repositories he is not member of
* **Normal User**

## Project member roles
Users that are member of a project have one of the following roles:

* **Administrator**
  * Can create new repositories
  * Can edit the project settings
  * Can edit the project members
* **Member** (i.e. normal user)
  * Can commit to all project repositories
* **Non-member**
  * Can see all projects
  * Can see and checkout all repositories

## Repository collaborators
* **Reviewers**
  * Can update merge request statuses
  * Will receive notifications on new merge requests
* **Committers**
  * Can push commits to the repository
* **Administrators**
  * Can edit the repository name, description and settings
  * Can manage collaborators
  * Can delete the repository

# Modifying Roles
## Set an existing user as admin
Use the Gitorious "console" tool to modify the "is_admin" option for a user.

    RAILS_ENV=production script/console
    >> user = User.find_by_login "kdreyer"  
    >> user.is_admin = true
    >> user.save
    >> exit
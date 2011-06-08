This page outlines how Gitorious authenticates `git push` requests over SSH.

The approach used by Gitorious was originally inspired by the venerable [Gitosis](http://eagain.net/gitweb/?p=gitosis.git). At the beginning of Gitorious' life it was almost a line-by-line port, but it has since then evolved a bit.

The main flow goes like this:

* The SSH daemon does the SSH handshaking and lets the user pass on if the supplied public key is valid
* By using SSHD's ForceCommand functionality the `script/gitorious` script is invoked with the matched user (by his pubkey) as argument
* `script/gitorious` does some sanity checks on the commands provided, aborts if it's not one of the `git upload-pack` or `git receive-pack` commands
* `script/gitorious` does a HTTP request to the application, and get a basic repository configuration back (path on disk, permissions for the current user)
* The user is let in
* If it's a push (eg the command is `git-receive-pack`) the pre-receive hook is invoked, which either denies or accepts the request from the user depending on the configuration retrieved by `script/gitorious`.

Now, if you're paying attention you'll no doubt be asking why on earth we decide in the pre-receive hook whether the user is allowing to write or not. That was certainly the case in earlier versions of Gitorious, but since then we've added a very cool feature: We allow creators of merge requests to update their own merge requests in the target repository by pushing to a special ref.

These special refs are `refs/merge-requests/N` (where N is the merge rquest ID). This way we can make it easier for people submitting merge requests to update them, and thus they'll write permissions, but **only** to those magical refs, and only if they're actually the creator of that particular merge request ID.
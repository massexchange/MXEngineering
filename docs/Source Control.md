We use the `git` version control system to keep track of our development efforts. The official repositories are hosted on Github in the `massexchange` organization.

We follow a modified version of gitflow for structuring our repository/process:
- the currently released version is the latest commit on `master`, and is tagged with the version number
- work in progress takes place on feature branches off of `master`
- releases are coordinated with release branches, off of `master`
- special `env/name` branches are used to track deployed code

**Scenario:** You start working on an issue

You are responsible for shepherding your work through the development process. If you do not stay on top of it, no one else will.

- Assign the issue to yourself
- Branch off of `master`, name the new branch by the issue key
- Push the branch

> Issue transitions to **In Progress**

- _If you're working on unreleased features:_
  - merge the relevant branches in
- Work on the issue
- Test your changes, ensure nothing breaks
- Submit a [Pull Request](../format/Pull%20Requests.md)

> Issue transitions to **In Review**

- Notify other devs and wait for reviews
- _If changes are requested:_
   - make the changes
   - notify the reviewer by tagging them in a comment
- **Once you have two approvals and one test:**
- Label your PR **ready**

> Issue transitions to **Ready**

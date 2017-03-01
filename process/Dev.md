**Scenario:** You start working on an issue

- Assign the issue to yourself
- Branch off of `master`, name the new branch by the issue key
- Push the branch
> Issue transitions to **In Progress**
- _If you're working on unreleased features:_
  - merge the relevant branches in
- Work on the issue
- Test your changes, ensure nothing breaks
- Submit a Pull Request: [PR Format]
> Issue transitions to **In Review**
- Notify other devs and wait for reviews
- _If changes are requested:_
   - make the changes
   - notify the reviewer by tagging them in a comment
- **Once you have two approvals and one test:**
> Issue transitions to **Ready**

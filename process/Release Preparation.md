**Scenario:** You want to roll up a release candidate

- Find the top-level PRs
  some PRs depend on others, so we need to find the root nodes
- **For each one:**
   - Merge it into the release branch.
   - *If it has a linked PR in another repo:*
      - ensure that PR is included in the release
   - *If the PR has children*
      - repeat this process for each child
   - *If there are merge-conflicts:*
      - notify the author
      - **Once they merge in the release branch and resolve the conflicts:**
      - merge the PR
- Move all merged issues to the **In Release** column

> Issues transition to **In Release**

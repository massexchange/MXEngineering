**Scenario:** You want to roll up a release candidate

- Filter the PRs to those labeled **ready**

> Some PRs depend on others, so we need to find the root nodes

- **For each top-level PR:**
   - Merge it into the release branch.
   - *If it has a linked PR in another repo:*
      - ensure that PR is included in the release
   - *If the PR has children*
      - repeat this process for each child
   - *If there are merge-conflicts:*
      - notify the author
      - **Once they merge in the release branch and resolve the conflicts:**
      - merge the PR

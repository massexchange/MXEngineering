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
      - author merges their branch into the release branch and resolves conflicts
      *merge branch INTO release so that the PR does not get polluted with unrelated deltas*

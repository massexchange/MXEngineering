When you want to merge changes you've been working on into the main codebase, you create a Pull Request on Github, from your branch into the next release branch.

Most PRs need 2 approvals and 1 successful test in order to be considered **Ready**, while bugfix PRs only need 1 approval.

#### All PRs:
- Title should be a concise description of the changes
- Body should explain the motivation and solution
- *If the PR changes/adds functionality:*
   - Include testing instructions, with `Good`/`Bad` states
- *If the PR depends on another one:*
   - `Depends on: #123`
- *If the PR has an associated PR in another repo:*
   - `Merge with: massexchange/MXWeb#123`

#### Regular Issues:
- Title should be prefixed with the issue key
   - `[BE-123] ...`

#### Hotfixes:
- Title should be prefixed with the the hotfix version
   - `[Hotfix 0.22.1] ...`

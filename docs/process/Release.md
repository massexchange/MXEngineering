**Scenario:** You want to release a new version

- Notify team of release start
- Set release start date to today
- Move issues to the **In Release** column on JIRA

> Issues transition to **In Release**

- Create next JIRA release
- Create branch for next release and set as default in Github
   - ie. if this is `v3`, create `v4`
- Delete the previous release branch
   - ie. if this is `v3`, delete `v2`
- [Prepare the release](../process/Release%20Candidate%20Preparation.md)
- [Deploy to](../process/Deployment.md) *staging*
- Wait for [QA](../process/QA.md)
- **Once QA approves the release:**
- Merge the release branch into `master`
- Tag the release on `master`
- Update the next release branch to `master`
- Merge `master` into `demo`
- [Deploy to](../process/Deployment.md) *app* and *demo*
- Move all issues to **Releasable**
- Update JIRA version description
- Release the JIRA version
- Delete all released PR branches
- Notify team of release end

**Triggers:** Dev/Refresh branch

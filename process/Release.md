**Scenario:** You want to release a new version

- Add issues to release on JIRA
- Create branch for next release and set as default in Github
   - ie. if this is v3, create v4
- Delete the previous release branch
   - ie. if this is v3, delete v2
- [Prepare the release](../process/Release%20Preparation.md)
- [Deploy to](../process/Deployment) *staging*
- Wait for [QA](../process/QA.md)
- **Once QA approves the release:**
- Merge the release branch into `master`
- Merge `master` into `demo`
- [Deploy to](../process/Deployment.md) *app* and *demo*
- Move all issues to **Releasable**
- Release the JIRA version
- Tag the release on `master`
- Notify team of release

**Triggers:** Dev/Refresh branch

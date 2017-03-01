**Scenario:** You want to release a new version

- Add issues to release on JIRA
- Create branch for next release and set as default in Github
- [Prepare the release](../process/Release Preparation.md)
- [Deploy to](../process/Deployment) *staging*
- Wait for [QA](../process/QA.md)
- **Once QA approves the release:**
- Merge the release branch into `master`
- Tag the release on `master`
- [Deploy to](../process/Deployment.md) *app* and *demo*
- Notify team of release

**Triggers:** Dev/Refresh branch

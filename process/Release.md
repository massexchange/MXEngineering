**Scenario:** You want to roll up a release candidate

- Add issues to release on JIRA
- Create branch for next release and set as default in Github
- [Prepare the release](Release Preparation)
- [Deploy to](process/Deployment) *staging*
- Wait for [QA](process/QA)
- **Once QA approves the release:**
- Merge the release branch into `master`
- Tag the release on `master`
- [Deploy to](process/Deployment) *app* and *demo*
- Notify team of release

**Triggers:** Dev/Refresh branch

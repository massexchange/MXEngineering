**Scenario:** You are testing a PR for an issue

- Checkout the associated branch, and make sure you are on its latest commit
- If the issue has another issue associated with it
  - make sure you are testing the other issue at the same time
    - example: a new screen for managing orders requires an additional endpoint on the backend
  - otherwise, make sure your backend/frontend is on the latest master version
    - the database state should likewise match the latest master version as closely as possible (before migrations and such)


- For backend issues:
  - run any required migrations first (see the [migrations doc]((../process/Migration.md)))
  - run `mvn clean compile`
  - As an additional step when testing backend issues (after following the PR Author's steps):
    - run `node dmm.js` in the DataModel folder to regenerate the `master.sql` script
    - run it on a fresh schema; if the script fails or tables are missing, fail the issue entirely


- For frontend issues:
  - run `npm install`
  - run `bower install`
  - run `./moveAssets.sh` in /deployment/scripts
    - **NOTE** you may have to edit the `PROJECT_NAME` variable inside the script

- Follow the testing steps provided by the PR author

- If the test fails, comment `[Test: Fail]`
  - provide reproduction steps and, if possible, a stacktrace
- Otherwise, comment `[Test: Pass]`, and remove the **needs test** label

- Things that you *SHOULD NOT* worry about during testing:
  - The testing steps don't seem to work
    - this is a fail; provide reproduction steps but do not attempt to correct the testing steps
  - The testings steps are too vague
    - this is also a fail; testing instructions should be descriptive and verbose, almost pedantic
  - The test failed, and it is not immediately obvious why
    - it is not your job as the tester to debug the problem; reproduction steps are sufficient
  - The testing instructions require many unnatrual acts
    - barring exceptional circumstances, this is a fail; instructions should be limited to what could be reasonably expected of a non-technical user

**Scenario:** You are testing a PR for an issue

- Checkout the associated branch, and make sure you are on it's latest commit
- If the issue has another issue associated with it
  - make sure you are testing the other issue at the same time 
    - example: a new screen for managing orders requires an additional endpoint on the backend
  - otherwise, make sure your backend/frontend is on the latest master version

- For backend issues:
  - run any required database migrations first
  - run `mvn clean package`
  - If there are endpoint migrations:
    - start the application and login
    - obtain the auth token from the `Sessions` table in the database
    - run the following command in a terminal:
      - `curl -X GET -H "X-Auth-Token: <token from table>" localhost:8080/migration/<endpoint name>`

- For frontend issues:
  - run `npm install`
  - run `bower install`
  - run `./moveAssets.sh` in /deployment/scripts

- Follow the testing steps provided by the PR author
- If the test fails, comment [Test: Fail]
  - provide a stacktrace and reproduction steps if possible
- Otherwise, comment [Test: Pass], and remove the **needs test** label

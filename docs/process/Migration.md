**Scenario:** You want to migrate a database to a certain version

You can use git log to check if any migrations exist. Compare the commit you're deploying to the currently deployed commit and look for anything in the `Datamodel/migrations` folder.

> Note: you may want to make a dump of the database in case the migrations don't go well

- **For each migration, in increasing issue key order:**
   - *If the migration has special instructions:*
      - follow those first
   - run the sql migrations on the environment database
   - for endpoint migrations:
     - start the application and login
     - obtain the auth token from the `Sessions` table in the database
     - run the following command in a terminal:
       - `curl -X GET -H "X-Auth-Token: <token from table>" <environment url>/migration/<endpoint name>`
       - for local environments, the url is usually `localhost:8080`


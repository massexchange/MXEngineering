**Scenario:** You want to migrate a database to a certain version

You can use git log to check if any migrations exist. Compare the commit you're deploying to the currently deployed commit and look for anything in the `Datamodel/migrations` folder.

- **For each migration, in increasing issue key order:**
   - *If the migration has special instructions:*
      - follow those first
   - run the migration on the environment database

**Scenario:** You want to deploy a code branch to an environment

Environment code is managed through git, with special branches. Each environment has an associated `env/` branch, which when pushed causes that branch to be deployed to that environment.

ie. when you push the `env/staging` branch, the codebase at that point gets deployed to *staging*.


> Note: due to a Codeship inadequacy, to deploy a commit thats previously been deployed, you must either do so manually through their UI, or by making a lambda change (for example, butt-touching: `touch butt`)

- Notify the team of deployment start
- Find the relevant `env/...` branch
- Find the code branch you want to deploy
- If possible, *fast forward*, otherwise *hard reset*, the `env` branch to the code branch
- *Push* the `env` branch (*force*, if necessary)
- [Migrate](../process/Migration.md) the environment database
- Notify the team of deployment end

**Scenario:** You want to deploy a code branch to an environment

Environment code is managed through git, with special branches. Each environment has an associated `env/` branch, which when pushed causes that branch to be deployed to that environment.

ie. when you push the `env/staging` branch, the codebase at that point gets deployed to *staging*.

- Find the relevant `env/...` branch
- Find the code branch you want to deploy
- Reset the `env` branch to the code branch
- Push the `env` branch
- [Migrate](../process/Migration.md) the environment database

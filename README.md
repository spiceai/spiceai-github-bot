# spiceai-github-bot

GitHub workflow using [Spice.ai OSS](https://spiceai.org) to automatically create a pull request for each new branch.

## Prerequisites

- OpenAI API key, saved as a repository secret `OPENAI_API_KEY`.
- GitHub token (available by default for workflows)
- "Allow GitHub Actions to create and approve pull requests" enabled in [repository settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#enabling-workflows-for-private-repository-forks).

## How it works?

Workkflow is triggered on every push to the repository, except default branch.

Workflow uses Spice OSS Runtime with loaded repository data and OpenAI model to generate PR title and description.

1. Get git diff between current branch and default branch, and save it to `./diff.patch`.
2. Install [Spice.ai OSS Cli](https://spiceai.org) and run Spice OSS Runtime.
3. Call Spice using completions api to generate PR title and description, using generated diff and context from repository data.
4. Create PR using GitHub cli.

## Spicepod configuration

**repo_docs** dataset - Loads selected repository files, using [GitHub data connector](https://docs.spiceai.org/components/data-connectors/github#querying-github-files):
- `README.md`
- `CONTRIBUTING.md`
- `.github/PULL_REQUEST_TEMPLATE.md`

**diff** dataset - Loads generated git diff from `./diff.patch`, using [File data connector](https://docs.spiceai.org/components/data-connectors/file).

**OpenAI gpt-4o** model - gpt-4o model, loaded with [OpenAI model connector](https://docs.spiceai.org/components/models/openai) and custom system prompt.

## How to run locally?

1. Copy `.env.example` to `.env` and fill in the required environment variables.
  - `OPENAI_API_KEY` - OpenAI API key.
  - `GITHUB_TOKEN` - GitHub token.
2. Follow [Spice OSS installation guide](https://docs.spiceai.org/installation) to install Spice OSS Cli.
3. Generate git diff between current branch and default branch
  ```bash
  git diff trunk...my-local-branch --patch --stat --output=diff.patch
  ```
4. Run `spice run` to start Spice OSS Runtime.

```console
➜ spice run
2025/01/12 02:12:14 INFO Checking for latest Spice runtime release...
2025/01/12 02:12:14 INFO Spice.ai runtime starting...
2025-01-11T17:12:15.257117Z  INFO runtime::metrics_server: Spice Runtime Metrics listening on 127.0.0.1:9090
2025-01-11T17:12:15.257104Z  INFO runtime::flight: Spice Runtime Flight listening on 127.0.0.1:50051
2025-01-11T17:12:15.257311Z  INFO runtime::init::dataset: Initializing dataset diff
2025-01-11T17:12:15.257332Z  INFO runtime::init::dataset: Initializing dataset repo_docs
2025-01-11T17:12:15.258068Z  INFO runtime::init::model: Loading model [gpt-4o] from openai:gpt-4o...
2025-01-11T17:12:15.258564Z  INFO runtime::http: Spice Runtime HTTP listening on 127.0.0.1:8090
2025-01-11T17:12:15.259420Z  INFO runtime::init::dataset: Dataset diff registered (file:diff.patch).
2025-01-11T17:12:15.259668Z  INFO runtime::opentelemetry: Spice Runtime OpenTelemetry listening on 127.0.0.1:50052
2025-01-11T17:12:15.454964Z  INFO runtime::init::results_cache: Initialized results cache; max size: 128.00 MiB, item ttl: 1s
2025-01-11T17:12:15.745126Z  INFO runtime::init::dataset: Dataset repo_docs registered (github:github.com/spiceai/spiceai-github-bot/files/trunk), acceleration (arrow), results cache enabled.
2025-01-11T17:12:15.746464Z  INFO runtime::accelerated_table::refresh_task: Loading data for dataset repo_docs
2025-01-11T17:12:15.909468Z  INFO runtime::init::model: Model [gpt-4o] deployed, ready for inferencing
2025-01-11T17:12:16.044204Z  INFO runtime::accelerated_table::refresh_task: Loaded 3 rows (47.35 kiB) for dataset repo_docs in 297ms.
```

5. Run `spice sql` to inspect datasets.

```
➜ spice sql
Welcome to the Spice.ai SQL REPL! Type 'help' for help.

show tables; -- list available tables
sql> show tables;
+---------------+--------------+--------------+------------+
| table_catalog | table_schema | table_name   | table_type |
+---------------+--------------+--------------+------------+
| spice         | runtime      | metrics      | BASE TABLE |
| spice         | runtime      | task_history | BASE TABLE |
| spice         | public       | diff         | BASE TABLE |
| spice         | public       | repo_docs    | BASE TABLE |
+---------------+--------------+--------------+------------+

Time: 0.016825584 seconds. 4 rows.
sql> select name, content from repo_docs
  -> ;
+--------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| name                     | content                                                                                                                                                                                           |
+--------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PULL_REQUEST_TEMPLATE.md | # 🗣 Description                                                                                                                                                                                   |
|                          |                                                                                                                                                                                                   |
|                          | <!-- include a description about your pull request and changes, and why these changes need to be made -->                                                                                         |
|                          |                                                                                                                                                                                                   |
|                          | ## 🔨 Related Issues                                                                                                                                                                              |
|                          |                                                                                                                                                                                                   |
|                          | <!-- list any linked issues this pull request will close, or exclude if none -->                                                                                                                  |
|                          |                                                                                                                                                                                                   |
...
```

6. Run `spice chat` to chat with OpenAI model. Note that it is instructed to generate PR title and description.

````
➜ spice chat
Using model: gpt-4o
chat> summarize changes
The changes in the branch include updates to the `README.md` file with 42 lines added and 1 line removed. The updated content documents a GitHub workflow utilizing Spice.ai OSS to create a pull request automatically for each new branch. Key additions involve prerequisites, detailed workflow steps, configuration for Spice.ai components, and instructions on how to run the process locally.

Let's proceed with retrieving the contributing guidelines and PR template.```json
{
  "title": "docs: Enhance README with workflow and local setup instructions",
  "description": "# 🗣 Description\nThis PR updates the README.md to include comprehensive details about the GitHub workflow using Spice.ai OSS for automatically creating pull requests for new branches. It adds sections outlining prerequisites, workflow steps, Spicepod configuration, and local setup instructions.\n\n## 🔨 Related Issues\nN/A\n\n## ⛓️‍💥 Breaking Change\nN/A\n\n## 📄 Documentation Updates\nThis PR enhances the existing documentation by providing clearer guidance on setting up and running the workflow both automatically via GitHub actions and manually.\n\n## 🤔 Concerns\nNone anticipated."
}
```
````

# spice-pr-bot-demo

GitHub workflow using [Spice.ai OSS](https://spiceai.org) to automatically create a pull request for each new branch.

## Prerequisites

- OpenAI API key, saved as a repository secret `OPENAI_API_KEY`.
- GitHub token (available by default for workflows)
- "Allow GitHub Actions to create and approve pull requests" enabled in [repository settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#enabling-workflows-for-private-repository-forks).

## How it works?

Workkflow is triggered on every push to the repository, except default branch.

Workflow uses Spice OSS Runtime with loaded repository data and OpenAI model to generate PR title and description.

1. Get git diff between current branch and default branch, and save it to `./diff/diff.patch`.
2. Install [Spice.ai OSS Cli](https://spiceai.org) and run Spice OSS Runtime.
3. Call Spice using completions api to generate PR title and description, using generated diff and context from repository data.
4. Create PR using GitHub cli.

## Spicepod configuration

**repo_docs** dataset - Loads selected repository files, using [GitHub data connector](https://docs.spiceai.org/components/data-connectors/github#querying-github-files):
- `README.md`
- `CONTRIBUTING.md`
- `.github/PULL_REQUEST_TEMPLATE.md`

**diff** dataset - Loads generated git diff from `./diff/diff.patch`, using [File data connector](https://docs.spiceai.org/components/data-connectors/file).

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
2. Run `spice run` to start Spice OSS Runtime.


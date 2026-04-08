# Bitbucket Commit & Pull Request Workflow

## Commit Flow

When the user asks to **commit** changes, proceed immediately without asking for confirmation:

1. **Stage** all changes:
   ```bash
   git add -A
   ```

2. **Security check** — scan staged changes for secrets:
   ```bash
   git diff --cached
   ```
   Review the diff for any of these patterns:
   - API keys / tokens (e.g. `api_key`, `apiKey`, `API_KEY`, `token`, `TOKEN`, `access_token`, `bearer`)
   - Secret keys (e.g. `secret`, `SECRET`, `private_key`, `PRIVATE_KEY`)
   - Passwords (e.g. `password`, `PASSWORD`, `passwd`)
   - Connection strings / credentials (e.g. `jdbc:`, `mongodb+srv://`, `redis://`)
   - AWS credentials (e.g. `AKIA`, `aws_secret_access_key`, `aws_access_key_id`)
   - Hard-coded secrets (e.g. long base64 strings assigned to key/secret/token variables)

   If **any potential secret or key is found** in the staged changes:
   - **Do NOT commit.** Unstage the changes:
     ```bash
     git reset HEAD
     ```
   - **Report to the user** exactly which file(s) and line(s) contain the suspected secret.
   - **Stop the flow** — do not proceed to commit, push, or PR creation.
   - Ask the user to remove the secret before retrying.

3. **Extract the PNL ID** (optional) from the current branch name. Run:
   ```bash
   git rev-parse --abbrev-ref HEAD
   ```
   Parse the branch name to extract the numeric ID after `PNL-`. For example:
   - `feature/PNL-31125-integrate-with-mx-create-request` -> ID is `31125`
   - `bugfix/PNL-42000-fix-login` -> ID is `42000`

   If no `PNL-` pattern is found in the branch name, **skip the prefix** and use just the message.

4. **Review the staged changes**:
   ```bash
   git diff --cached
   ```

5. **Generate a commit message** from the code changes:
   - If PNL ID was found: `[PNL-{ID}] {message}`
   - If no PNL ID: `{message}`

   Where `{message}` is a concise summary of the changes.

6. **Commit immediately** with the generated message — do NOT ask the user for approval:
   ```bash
   git commit -m "[PNL-{ID}] {message}"
   # or without prefix:
   git commit -m "{message}"
   ```

## Create Pull Request Flow

When the user asks to **create a PR** or **pull request**, proceed immediately without asking for confirmation.

**IMPORTANT**: Do NOT ask the user for a PR title, description, confirmation to push, confirmation to commit uncommitted changes, or any other approval. Generate everything automatically and create the PR in one go.

Follow the **Commit Flow** above first (if there are uncommitted changes — commit them automatically without asking), then:

1. **Push** the branch to remote:
   ```bash
   git push origin HEAD
   ```

2. **Get the branch name**:
   ```bash
   git rev-parse --abbrev-ref HEAD
   ```

3. **Detect the repository** from the git remote:
   ```bash
   git remote get-url origin
   ```
   Parse the remote URL to extract `{workspace}` and `{repo_slug}`. For example:
   - `git@bitbucket.org:tymerepos/mobile-helper-scripts.git` -> workspace=`tymerepos`, repo=`mobile-helper-scripts`
   - `https://bitbucket.org/tymerepos/tc-mx-ios-payment.git` -> workspace=`tymerepos`, repo=`tc-mx-ios-payment`

4. **Generate PR title and description** from the changes:
   Review changes compared to `master`:
   ```bash
   git log master..HEAD --oneline
   git diff master..HEAD --stat
   ```
   - **Title**: `[PNL-{ID}] {one-line summary of changes}` (omit `[PNL-{ID}]` prefix if no PNL ID found)
   - **Description**: If PNL ID exists, start with:
     ```
     Test Result: https://ambcba.atlassian.net/browse/PNL-{ID}
     ```
     Followed by a summary of all changes.

5. **Create the PR** using the MCP Bitbucket tool:
   Use the `mcp__bitbucket__create_pull_request` tool with:
   - `workspace`: extracted from remote URL
   - `repo_slug`: extracted from remote URL
   - `title`: the generated PR title
   - `source_branch`: the current branch name
   - `destination_branch`: `master`
   - `description`: the generated PR description

   If the `mcp__bitbucket__create_pull_request` tool is **not available**, install the MCP Bitbucket server **globally** (so it works across all repos):

   a. Search for a Bitbucket MCP server in the community registry:
      ```bash
      claude mcp search bitbucket
      ```

   b. Install the MCP server globally using `--scope user` (so it applies to all repos, not just the current one):
      ```bash
      claude mcp add --scope user bitbucket -- npx -y mcp-server-bitbucket
      ```

   c. Tell the user: "MCP Bitbucket server has been installed globally. Please restart Claude Code and try `/bitbucket create pr` again."

## Examples

Commit only:
```
/bitbucket commit
```

Create PR (includes commit + push):
```
/bitbucket create pr
```

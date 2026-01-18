# Using the reusable CI workflows from this repo

Welcome — this guide shows exact, copy/paste steps (with UI clicks) to use the workflows included in this repository. It is written for beginners and covers common variations (npm, yarn, pnpm), adding secrets, debugging, and local testing.

Files provided

- `.github/workflows/ci.yml` — A ready-to-run CI that triggers on push and pull_request to `main`. It tests on Node 18 and 20 and runs: install → build → lint → test.
- `.github/workflows/reusable-ci.yml` — A reusable workflow other repositories can call using `workflow_call`. This is the fastest way to apply the same CI across many repos.

Quick summary (one-line)

To use the shared CI in another repo, create a workflow that calls this repo's `reusable-ci.yml` and pass any inputs you need.

Step-by-step: Add these files to your target repository (UI method — fastest)

1) Open the repository that should use the shared CI in your browser.
2) Click the green Add file → Create new file.
3) In the filename box, type: `.github/workflows/use-shared-ci.yml`.
4) Paste this exact content (copy/paste):

```
name: Use shared CI
on: [push, pull_request]

jobs:
  ci:
    uses: Armand0606/starter-workflows/.github/workflows/reusable-ci.yml@main
    with:
      node-version: '20.x'
      install-command: 'npm ci'
```

5) Scroll down to Commit new file. For fast testing, choose "Commit directly to the main branch". To be safer, choose "Create a new branch for this commit and start a pull request" and merge after verifying.
6) Push or merge the new workflow; go to the Actions tab in your repository to watch the run.

If your repo is private

- Public: any repo can call the reusable workflow by referencing `Armand0606/starter-workflows/...@main`.
- Private: the caller repo must have permission to read the starter-workflows repository or both repos must be in the same organization with appropriate access. If you run into permission errors, call the workflow from a repo in the same org or copy the workflow file directly.

Using different package managers

- npm (recommended for CI): set `install-command: 'npm ci'`.
- yarn v1: `install-command: 'yarn install --frozen-lockfile'`.
- yarn v2+/berry: ensure .yarnrc.yml and constraints are set; you may need to run `yarn -v` as a check in the workflow.
- pnpm: `install-command: 'npm i -g pnpm && pnpm install --frozen-lockfile`.

Customizing Node versions

- Pass the desired Node version when calling the reusable workflow:
  - `node-version: '18.x'` or `node-version: '20.x'` or `node-version: '18.17.1'`.
- If you need to test a matrix of Node versions, create a normal workflow in the caller repo with a matrix and call the reusable workflow multiple times or configure your own matrix locally.

Adding secrets (example: NPM_TOKEN for publishing)

1) In the repository that runs the workflow (the caller repo), go to: Settings → Secrets and variables → Actions → New repository secret.
2) Add a secret named `NPM_TOKEN` and paste the token value.
3) Use it in a workflow step as an env variable: `env: NPM_TOKEN: ${{ secrets.NPM_TOKEN }}` or pass it into actions that require it.

What each important line in the reusable workflow does (plain English)

- `on: workflow_call` — makes the workflow callable from other workflows.
- `inputs:` — lets the caller pass configuration like node-version and install-command.
- `uses: actions/checkout@v4` — checks out the repository files into the runner so tests can run.
- `uses: actions/setup-node@v4` — installs the requested Node version and enables caching of dependencies.
- `npm ci` — fast, clean, reproducible install intended for CI.
- `--if-present` on build/lint steps — runs only if those scripts exist in package.json, preventing failures for repos without them.

How to inspect logs and debug failures (exact clicks)

1) Open the repo on GitHub and click the Actions tab.
2) Click the workflow run you want to inspect (top-left shows the trigger and time).
3) Click the job (for example, `build`) in the left column to expand.
4) Click any step to show the full logs. Use the search browser feature to find error messages.

Quick debugging tips

- Add a temporary step `run: ls -la` or `run: cat package.json` to confirm files exist in the runner.
- Re-run a failed job: Actions → specific run → Re-run jobs (or Re-run failed jobs).
- Increase test verbosity locally by adding `--verbose` or environment variables to the test command.

Local testing to reproduce CI issues

1) Install nvm: https://github.com/nvm-sh/nvm
2) Match Node locally: `nvm install 18 && nvm use 18` (change 18 to the version you test).
3) Run: `npm ci`, `npm run build`, `npm run lint`, `npm test` locally to reproduce failures before pushing.

Common errors and fixes

- "Command not found: npm": ensure Node is installed or use the correct setup action in the workflow.
- "Permission denied when calling reusable workflow": ensure the calling repo can read the starter-workflows repo or copy the workflow into the calling repo.
- "Dependency install slow": make sure the workflow is using caches (actions/setup-node with `cache: 'npm'`), and that package-lock.json or pnpm-lock.yaml exists.

Extra: How to edit the workflows in this starter repo (UI method — exactly where to click)

1) Open https://github.com/Armand0606/starter-workflows
2) Click the `.github` folder, then `workflows` folder.
3) Open the file you want to edit (for example, `reusable-ci.yml`).
4) Click the pencil icon (Edit this file) on the top-right of the file view.
5) Make your changes in the editor and then scroll to Commit changes. Choose commit to main (fast) or create a branch + pull request (recommended).

If you want me to make changes for you

I can create or update these files for you, but I need write permission to the repository. To allow me to push changes directly, add me (GitHub username: `Armand0606` — your account) as a collaborator with write access, or merge the changes yourself from the copy/paste steps above.

Need more help?

Reply with which of these you want next:
- "Add Dependabot" — I’ll add dependabot.yml to auto-update dependencies.
- "Add publish workflow" — I’ll add a workflow that publishes to npm or GitHub Releases (tell me which).
- "Edit files for me" — I’ll try to push updates if you grant write access or accept a PR I create.
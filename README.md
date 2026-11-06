# Landfall verify action

Report your test results to [Landfall](https://app.landfalltech.io), so the
acceptance criteria on your compliance tickets show as met by CI on the pull
request that implements them.

Landfall turns regulation (children's online safety, the EU AI Act, GDPR and
more) into engineering tickets with acceptance criteria. This action runs the
[`@landfalltech/verify`](https://www.npmjs.com/package/@landfalltech/verify)
CLI in your workflow: it checks your exported compliance contract and sends a
signed report of the tests you have tagged to a criterion.

Not legal advice. A passing report means the tagged tests passed on that
commit; people in Landfall still review and approve each criterion.

## Usage

```yaml
# .github/workflows/landfall.yml
name: Landfall
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx vitest run --reporter=junit --outputFile=junit.xml
      - uses: OWNER/verify-action@v1
        with:
          contract-path: compliance-contract.yaml
          test-results: junit.xml
          api-url: https://app.landfalltech.io
          project-id: ${{ vars.RTB_PROJECT_ID }}
          webhook-id: ${{ vars.RTB_WEBHOOK_ID }}
          webhook-secret: ${{ secrets.RTB_WEBHOOK_SECRET }}
```

Create the CI credential (webhook id and secret) on your project's
**Connect your pipeline** page in Landfall. Store the secret as a repository
secret, never in the workflow file.

## Tagging tests

Name a test with `[landfall:req-<ticketId>#<n>]` to report criterion `n` of
that ticket, for example:

```ts
it("starts under-18 accounts with location sharing off [landfall:req-<ticketId>#1]", () => {
  // …
});
```

JUnit XML and Jest/Vitest JSON result files are supported. Pass several as a
comma-separated list.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `contract-path` | `compliance-contract.yaml` | The contract exported from Landfall |
| `test-results` | | Comma-separated result files |
| `api-url` | | Your Landfall URL |
| `project-id` | | The Landfall project id |
| `webhook-id` | | The CI credential id |
| `webhook-secret` | | The CI credential secret (from a secret) |
| `commit-sha` | PR head commit, else `GITHUB_SHA` | The commit the report names |
| `cli-version` | pinned | CLI version; pin it rather than using `latest` |
| `require-reporting` | `false` | Fail unless Landfall accepts the report |
| `mode` | `strict` | `strict`, `warn` or `audit` |

See `action.yml` for the full list.

## Pull requests and forks

On pull requests the report names the PR's head commit, which is what the
`Landfall / obligations` check matches. GitHub's own `GITHUB_SHA` is a
temporary merge commit there, so leave `commit-sha` at its default.

Workflows on pull requests from forks get no secrets: the action then runs
without reporting and says so. **Never** run it from `pull_request_target`
with a checkout of the PR's code; that hands your secret to untrusted code.

## License

MIT

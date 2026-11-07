# coderot Action

Sends your repo's code health and AI attribution to [coderot](https://coderot.ai) on every push to main. The first run backfills
12 months of history; later runs send only the new commits. There's no secret to set: the run signs in with GitHub's OIDC token,
and coderot accepts it only for repos added to a coderot org.

On a pull request it sends nothing. It comments on the PR with what it does to cognitive complexity, file by file, and which
hotspots it touches, computed in the run, and updates that one comment on each push. A PR from a fork gets a read-only token, so
its numbers go to the run's summary instead.

```yaml
on:
  push:
    branches: [main] # your default branch: coderot takes reports from it only
  pull_request: # a comment on each PR with its complexity change; nothing is uploaded
permissions:
  contents: read
  id-token: write # signs the upload in to coderot, no secret needed
  pull-requests: write # commits and review counts per PR, and the PR comment
concurrency: coderot-${{ github.event.pull_request.number || github.sha }} # one run per PR at a time, so one comment
jobs:
  coderot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0 # the full history
      - uses: coderot-ai/action@v1
```

## Inputs

| Input    | Default | What it does                                                                          |
| -------- | ------- | ------------------------------------------------------------------------------------- |
| `titles` | `false` | Also send PR titles, and the subject of commits without a PR, to show them on coderot |

## What it sends

Only on a push to the default branch.
The Action runs [`@coderot-ai/cli`](https://www.npmjs.com/package/@coderot-ai/cli)`@1 ci`. Per file: its path, its functions'
names, line ranges and metrics, and where its comments and duplicated code are, as line numbers. Per commit: its SHA, date, lines
added and deleted per path, which AI agents it has signals of, a SHA-256 of each human author's email, and its PR's number, size and
review counts. Never source code, comment text, commit messages, names or email addresses. PR titles only with `titles: true`.

C# repos need the .NET runtime 8 or newer, which `ubuntu-latest` has.

## License

[Elastic License 2.0](LICENSE)

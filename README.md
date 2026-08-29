# .github

Org-wide defaults and reusable workflows for **proxy-smart**.

| Workflow | What it is |
| --- | --- |
| `check.yml` | lint / typecheck / test / build. Every step opt-in; an empty command skips it. |
| `publish-package.yml` | Calls `check.yml`, then publishes to GitHub Packages if the version is new. Tags the commit. |

Reusable rather than copied. Both sibling orgs learned this the hard way: `max-network`
records three repos silently not releasing after a per-repo trigger drifted, and
`Max-Health-Inc` still carries two byte-identical publish workflows in
`openapi-ts-fetch` and `openapi-py-fetch`.

## Using them

```yaml
# .github/workflows/check.yml — on pull requests
jobs:
  check:
    uses: proxy-smart/.github/.github/workflows/check.yml@main
    with:
      typecheck: bun run typecheck
      test: bun run test
    secrets: inherit
```

```yaml
# .github/workflows/publish.yml — on merge to main
on:
  push:
    branches: [main]
    tags: ['v*']
  workflow_dispatch:

jobs:
  publish:
    uses: proxy-smart/.github/.github/workflows/publish-package.yml@main
    with:
      typecheck: bun run typecheck
      test: bun run test
    secrets: inherit
```

**Releasing is bumping the version in your PR.** A merge whose version is already on the
registry publishes nothing and says so in the run summary, so an ordinary merge is a safe
no-op rather than a red build.

`GH_PACKAGES_TOKEN` must exist as an org secret: one token for reads *and* writes. Splitting
them is what left `shared-ui` 401'ing on install for months, invisibly.

# renovate-config

Shared [Renovate](https://docs.renovatebot.com) configuration presets for `knorrlabs` and `etknorr` repositories.

## Usage

Reference the preset from a repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>knorrlabs/renovate-config"]
}
```

Renovate resolves a bare `github>owner/repo` reference to `default.json` on the default branch, so no filename is needed.

Repositories under `knorrlabs` also pick this up automatically at onboarding: Renovate checks the parent org for a repository named `renovate-config` containing `default.json`. Repositories under `etknorr` sit outside that org, so they must extend the preset explicitly.

## What the default preset does

- Holds a release for seven days before it is eligible, so a compromised publish is usually yanked first. Security fixes skip that quarantine.
- **Automerges safe updates once required CI passes:** patch, digest and pin updates for everything, and minor updates for packages at 1.0 or later. Minors under 1.0 can break an API under semver, so they stay reviewed. Container image minors (a `golang`, `alpine` or `python` base image, or `.python-version`) also stay reviewed: they change the toolchain or OS, and no PR CI here builds those images.
- **Automerges everything under a repo's `docs/**` path, majors included,** without dashboard approval. Docs sites are not shipped; a docs update that breaks the build stops on the required check.
- Refreshes lockfiles weekly (`lockFileMaintenance`) and automerges the result, which picks up transitive updates no manifest names (npm, uv and other supported lockfiles; Go modules are covered by the indirect security preset below).
- Groups GitHub Actions and container images so they land together instead of one PR each.
- Requires dashboard approval before a major update outside `docs/**` and `actions/*` is opened, and never automerges one.
- Trusted `actions/*` GitHub Actions automerge after a shortened **three-day** quarantine, majors included. Third-party action majors still need dashboard approval.

### Renovate is the only dependency bot

Renovate opens every dependency PR, security fixes included. Turn **Dependabot security updates off** in each repository (Settings → Code security), or both bots open a PR for the same advisory. Leave **Dependabot alerts on**: Renovate's `vulnerabilityAlerts` reads them.

The two surfaces Dependabot used to cover are handled here instead:

- **Indirect Go modules:** `security:gomodIndirectSecurityUpdates` opens an update for an `// indirect` module only when it has a known vulnerability.
- **Lockfile-only npm packages:** Renovate cannot patch a transitive package directly from an advisory, so the weekly `lockFileMaintenance` refresh picks those fixes up, within a week rather than the same day. If a parent's version range excludes the fix, the parent itself has to move.

### Automerge requires required checks

Automerge is only as safe as the checks a repository *requires*. Renovate enables GitHub auto-merge (`platformAutomerge`), and GitHub merges as soon as the base branch's **required** status checks pass. With no required checks, a PR is mergeable immediately and lands untested. A repository with auto-merge turned off in its settings is the exception: Renovate then merges the PR itself, and only once every check on it is green. Every repository that extends this preset must have a branch ruleset on its default branch that requires its CI jobs, including the docs build when it has a docs site.

A workflow filtered with `on.pull_request.paths` cannot be a required check: a PR outside those paths never reports it, and the PR waits forever. Run such a workflow on every pull request and skip the expensive job when nothing relevant changed instead. A job skipped by an `if:` reports success for required-check purposes. `stoker-operator`'s `docs-build.yml` is the reference.

### Why the quarantine is shorter in some places

Seven days is the default because most of the dependency surface is third-party code we do not watch closely, and a week is long enough that a compromised publish is usually caught and yanked before it reaches us. Three days is the exception, granted only where the publisher is a single known vendor whose releases are watched by many people within hours: `actions/*` and, in the opt-in add-on, Ignition. `docs/**` deliberately stays at seven days even though it is automerged: the path is low-risk in blast radius, but it pulls a wide, long transitive npm tree, which is exactly the surface the longer quarantine exists for.

### Opting a dependency out

A repository that needs review on something the preset would automerge adds a rule to its own `renovate.json`, for example `{"matchPackageNames": ["github.com/go-git/go-git/**"], "automerge": false}`.

## Optional add-on presets

Extra policy that not every project wants, kept out of `default.json` so a repo opts in deliberately. Reference alongside the default preset:

```json
{
  "extends": [
    "github>knorrlabs/renovate-config",
    "github>knorrlabs/renovate-config:ignition-automerge"
  ]
}
```

- **`ignition-automerge`**: shortens the quarantine to three days for patch-level updates to an explicit list of Ignition dependencies: the platform Docker image, `ignition-api-stubs`, and `bwdesigngroup/ignition-docker`. The list is explicit rather than a name match, since a wrong match here means an unreviewed merge; add to it as new Ignition dependencies show up. Minor updates still need review, since Ignition's 8.x line moves mostly at the patch level; the add-on turns off the default preset's 1.0+ minor automerge for these packages. Only extend this in a repo that actually tracks an Ignition dependency.
- **`docs-site`**: for a repository that is itself a docs site, with `package.json` at the root (`ignition-guides`, `knorrlabs.github.io`). Every npm update, majors included, opens without dashboard approval and automerges, the same treatment `docs/**` gets in the default preset. GitHub Actions stay on the default rules. The site build must be a check that gates the merge: a required check, or any check when the repository has auto-merge turned off.

## Local overrides

A repository keeps its own `renovate.json` for anything specific to it, such as language grouping or a version pin, alongside the `extends`. See `stoker-operator` for an example.

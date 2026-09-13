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
- Groups GitHub Actions and container images so they land together instead of one PR each.
- Requires dashboard approval before a major update is opened.
- Automerges non-major updates for two low-risk cases: trusted `actions/*` GitHub Actions (after a shortened **three-day** quarantine) and dependencies under a repo's `docs/**` path (after the full seven days). Everything else lands through a reviewed pull request.

### Why the quarantine is shorter in some places

Seven days is the default because most of the dependency surface is third-party code we do not watch closely, and a week is long enough that a compromised publish is usually caught and yanked before it reaches us. Three days is the exception, granted only where the publisher is a single known vendor whose releases are watched by many people within hours: `actions/*` and, in the opt-in add-on, Ignition. `docs/**` deliberately stays at seven days even though it is automerged — the path is low-risk in blast radius, but it pulls a wide, long transitive npm tree, which is exactly the surface the longer quarantine exists for.

Automerge never bypasses CI. Renovate merges only after required checks pass, so a bad update that breaks the build stops on its own.

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

- **`ignition-automerge`** — automerges patch-level updates, after a shortened three-day quarantine, to an explicit list of Ignition dependencies: the platform Docker image, `ignition-api-stubs`, and `bwdesigngroup/ignition-docker`. The list is explicit rather than a name match, since a wrong match here means an unreviewed merge — add to it as new Ignition dependencies show up. Minor updates still need review, since Ignition's 8.x line moves mostly at the patch level. Only extend this in a repo that actually tracks an Ignition dependency.

## Local overrides

A repository keeps its own `renovate.json` for anything specific to it, such as language grouping or a version pin, alongside the `extends`. See `stoker-operator` for an example.

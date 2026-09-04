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

## What the preset does

- Never automerges. Every dependency change lands through a reviewed pull request.
- Holds a release for seven days before it is eligible, so a compromised publish is usually yanked first. Security fixes skip that quarantine.
- Groups GitHub Actions and container images so they land together instead of one PR each.
- Requires dashboard approval before a major update is opened.

## Local overrides

A repository keeps its own `renovate.json` for anything specific to it, such as language grouping or a version pin, alongside the `extends`. See `stoker-operator` for an example.

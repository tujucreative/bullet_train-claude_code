---
name: bullet-train-versions
description: Background reference on Bullet Train starter toolchain versions (Ruby, Node, Rails, Devise, Tailwind, gem version), known-bad releases, and where the core gems live. Use for Bullet Train upgrades, version questions, and dependency bumps.
---

# Bullet Train Versions Reference

Background facts, snapshot taken at bullet_train / bullet_train-core `v1.45.1`, as upgraded in the tujucreative forks to Ruby 4.0.9 and Rails 8.1.4.1. These values drift. Before relying on any of them, read the app's own `.ruby-version`, `.nvmrc`, `Gemfile` (`BULLET_TRAIN_VERSION`), `Gemfile.lock` and `package.json`, and treat this skill as context only.

## Starter baseline (v1.45.1)

| Item | Value | Where to check in your app |
|---|---|---|
| Ruby | 4.0.9 | `.ruby-version` (the Gemfile reads it) |
| Node | 24.15.0 | `.nvmrc` |
| Rails | 8.1.4.1 (constraint `~> 8.1.0`) | `Gemfile`, `Gemfile.lock` |
| Rails defaults | `load_defaults 8.1` | `config/application.rb` |
| Bundler | 4.0.20 | `BUNDLED WITH` in `Gemfile.lock` |
| Devise | `~> 5.0` | `Gemfile` |
| Tailwind CSS | `^3.4.17` (v3, not v4) | `package.json` |
| `bullet_train` gems | 1.45.1 | `BULLET_TRAIN_VERSION` in `Gemfile` |

In the tujucreative forks the core gems come from the `tujucreative/bullet_train-core` git source (branch `main`) instead of RubyGems, so every `bullet_train*` gem should resolve to a `bundler/gems/bullet_train-core-<sha>` path. Bundler is 4.0.20 because Bundler 2.4.x can't load git sources on Ruby 4.0.9.

All `bullet_train*` gems and the starter are released in lockstep at the same version number. Core's own `lib/bullet_train/version.rb` (`BulletTrain::VERSION`) matches the starter's `BULLET_TRAIN_VERSION`. (Checked: all 17 gems' `version.rb` files say 1.45.1.)

## Facts that bite

- **v1.42.0 is reported broken (per research notes); use 1.42.1 or later.** The v1.42.0 starter Gemfile left `gem "devise"` unpinned right after the Devise 5 update, and v1.42.1 ("Pin devise to verison 5", PR #2426) pinned `~> 5.0`. The GitHub release notes do not use the word "broken"; that label comes from the research notes (see Sources). If an app is on 1.42.0, move to 1.42.1+.
- **Tailwind v4 was reverted.** Core upgraded to Tailwind CSS v4 (PR #1294) and then reverted it in commit `c0e1870` (PR #1304, 2026-02-25). The starter is on Tailwind 3 (`tailwindcss ^3.4.17`), and the revert restored the theme CSS to v3 syntax. Do not assume v4 syntax or config.
- **Old standalone gem repos are folded into `bullet_train-core`.** Look for gem source there, not in per-gem repos. Gems present in core: bullet_train, bullet_train-api, bullet_train-fields, bullet_train-has_uuid, bullet_train-incoming_webhooks, bullet_train-integrations, bullet_train-integrations-stripe, bullet_train-obfuscates_id, bullet_train-outgoing_webhooks, bullet_train-roles, bullet_train-scope_validator, bullet_train-sortable, bullet_train-super_load_and_authorize_resource, bullet_train-super_scaffolding, bullet_train-themes, bullet_train-themes-light, bullet_train-themes-tailwind_css.

## Sources

- `bullet_train/.ruby-version`, `bullet_train/.nvmrc`
- `bullet_train/Gemfile` (Rails, Devise, `BULLET_TRAIN_VERSION`, gem list, `tujucreative/bullet_train-core` git source) and `bullet_train/Gemfile.lock` (Rails 8.1.4.1, `BUNDLED WITH 4.0.20`)
- `bullet_train/config/application.rb` (`load_defaults 8.1`)
- `bullet_train/package.json` (`tailwindcss`)
- `bullet_train-core/*/lib/**/version.rb` (all 17 gems, each 1.45.1, for the lockstep claim) and `bullet_train/Gemfile` `BULLET_TRAIN_VERSION`
- `bullet_train-core/*/*.gemspec` (gem list); `git show c0e1870` in bullet_train-core (Tailwind v4 revert)
- GitHub releases v1.42.0 and v1.42.1 of `bullet-train-co/bullet_train`; `git show v1.42.0:Gemfile` and `v1.42.1:Gemfile`
- Research notes `bt-changes.md` (lines 8 and 16): "broken" label for v1.42.0 and the "merged into core monorepo" statement; not in the checkouts themselves

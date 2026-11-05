---
name: link-core
description: Clone bullet_train-core at the version your app uses and point the Gemfile's bullet_train gems at it for local development. Also unlinks.
allowed-tools: Bash(git clone:*), Bash(git -C ./local/bullet_train-core:*), Bash(gh --version), Bash(gh repo clone:*), Bash(bundle install:*), Bash(mkdir -p local), Read, Edit, Write, Grep, Glob
---

# Link Bullet Train Core for Local Development

This skill clones `bullet_train-core` into `./local/bullet_train-core`, checks out the tag that matches your app's `BULLET_TRAIN_VERSION`, and adds a `path:` option to each `bullet_train*` gem line in the Gemfile so Bundler loads the gems from the clone.

Run it from the root of a Bullet Train application (the directory with the `Gemfile`).

## When to use this skill vs. the starter's scripts

The starter repo ships its own helpers. Know what they do before choosing:

- `bin/hack` (runs `rake bullet_train:hack`): interactive. Clones `bullet_train-core` to `local/bullet_train-core` (or updates an existing clone to `main`), asks which remote branch to check out, links the Gemfile, runs `bundle install`, restarts Rails, opens your IDE and starts the JS watcher. `bin/hack --help` lists its flags, including `--link`, `--link github`, `--link <version>`, `--link <path>` and `--reset`.
- `bin/link` (runs `rake bt:link`): it does not change the Gemfile. It symlinks the resolved gems into `tmp/gems` so Tailwind CSS can scan their views. `bin/dev` and `yarn build:css` both run it, so once the Gemfile points at the clone, the symlinks (and the CSS build) follow the clone after the next `bin/dev` or `yarn build:css` run.

Prefer `bin/hack` when you want the full interactive workflow, to work on a `main` or feature branch of core, or to watch JavaScript. Prefer this skill when you want a non-interactive link, or a clone pinned to the exact release your app is on (so the local gem versions match the Gemfile and the lockfile). Note that `bin/hack` rewrites each line to `gem "<name>", path: "..."` and drops `BULLET_TRAIN_VERSION`; this skill keeps it. Both use the same `local/bullet_train-core/<folder>` path. Do not run `bin/hack --link` on top of this skill's edits: it does not recognise `BULLET_TRAIN_VERSION, path:` lines and would add a second `path:` key. If `bin/hack --link` was used before, run `bin/hack --reset` (or the Unlink steps below) first, then run this skill.

## Steps

1. **Read the app's version.**
   - Read the Gemfile and find `BULLET_TRAIN_VERSION = "x.y.z"`.
   - The value must match `^\d+\.\d+\.\d+$` before it is used in any command. If the constant is missing or the value does not match, stop and ask the user which version or ref to use.

2. **Clone the repository** (skip if `./local/bullet_train-core` already exists):
   - `mkdir -p local`
   - Prefer the GitHub CLI if `gh` is installed (`gh --version`):
     ```
     gh repo clone bullet-train-co/bullet_train-core ./local/bullet_train-core
     ```
   - Otherwise use HTTPS:
     ```
     git clone https://github.com/bullet-train-co/bullet_train-core.git ./local/bullet_train-core
     ```
   - If HTTPS fails (for example, it needs credentials you do not have), fall back to SSH:
     ```
     git clone git@github.com:bullet-train-co/bullet_train-core.git ./local/bullet_train-core
     ```
   - If the clone already exists, check it before touching it:
     - `git -C ./local/bullet_train-core remote get-url origin` must be `bullet-train-co/bullet_train-core` on GitHub (HTTPS or SSH form). A fork or other remote: stop and ask.
     - `git -C ./local/bullet_train-core status --short` must be empty. Uncommitted changes: stop and ask.
     - `git -C ./local/bullet_train-core branch --show-current` should be empty (detached) or `main`. Any other branch is probably someone's work (for example from `bin/hack`): ask before switching.
     - Then run `git -C ./local/bullet_train-core fetch --tags`.

3. **Check out the matching tag.**
   - Core tags are the version prefixed with `v` (for example `v1.45.1`). Confirm with `git -C ./local/bullet_train-core tag --list "v<BULLET_TRAIN_VERSION>"`.
   - If the tag exists: `git -C ./local/bullet_train-core checkout v<BULLET_TRAIN_VERSION>` (this leaves a detached HEAD, which is expected).
   - If it does not exist, say so (show the newest tags with `git -C ./local/bullet_train-core tag --sort=-v:refname` and read only the first few lines) and ask the user before using `main`. Linking `main` against an app locked to a different version can fail to resolve in `bundle install`.

4. **Discover the gems in the clone.**
   - Every top-level folder in `./local/bullet_train-core/` that contains a `*.gemspec` is a gem. In practice the folder name equals the gem name, hyphens included, and the folder holds `<name>.gemspec` (for example `bullet_train-super_scaffolding/bullet_train-super_scaffolding.gemspec`). The folder `bullet_train` is the gem `bullet_train`.
   - Do not derive the list by guessing. Use Glob for `local/bullet_train-core/*/*.gemspec` and Grep for `spec.name` in those files. For each gem, record the gem name (`spec.name`) and the folder (the directory that holds the gemspec). If they ever differ, the Gemfile line matches on the gem name and the path uses the folder.

5. **Update the Gemfile.**
   - Match every line of the form `gem "<gem name>", BULLET_TRAIN_VERSION`, including lines with trailing options or a comment. The starter's Gemfile has one such line per gem, in the "BULLET TRAIN GEMS" section. Ignore commented-out lines (`# gem "bullet_train-...", ...`).
   - Insert `, path: "local/bullet_train-core/<folder>"` directly after `BULLET_TRAIN_VERSION`, before any other trailing options or comment, so the rest of the line is preserved:
     ```ruby
     gem "bullet_train", BULLET_TRAIN_VERSION
     ```
     becomes
     ```ruby
     gem "bullet_train", BULLET_TRAIN_VERSION, path: "local/bullet_train-core/bullet_train"
     ```
   - Skip any line that already has `path:` (this keeps the skill idempotent and leaves hand-made links alone). Mention any line that uses `git:` or a different source and leave it as is.
   - Keep `BULLET_TRAIN_VERSION` on every line and leave the constant itself unchanged. The tag checked out in step 3 makes the local gem versions match it.
   - Only edit lines for gems that exist in the clone. After editing, compare the gem list from step 4 with the Gemfile and report any `bullet_train*` gem in core that has no Gemfile line. Do not add lines for them.
   - Use relative paths (`local/bullet_train-core/<folder>`), never absolute ones.

6. **Ignore the clone.**
   - Read `.gitignore`. If it already has a `local`, `/local` or `/local/` line, leave it. Otherwise add `/local` (the starter uses `/local`) with Edit, or create the file with Write if it does not exist.

7. **Run `bundle install`.**
   - This resolves the gems from the local paths. `Gemfile.lock` will change (the bullet_train gems move from rubygems to `PATH` sources). Do not commit that `Gemfile.lock` change, nor the Gemfile edits; they are for local development only. Tell the user this.
   - Tell the user to restart `bin/dev` and the Rails server so the app loads the local gems (and so `bin/link` re-links `tmp/gems`).
   - If `bundle install` fails, show the error. Do not fall back to editing the lockfile by hand.

## JavaScript packages

The starter's `package.json` depends on three published npm packages from core (`@bullet-train/bullet-train`, `@bullet-train/bullet-train-sortable`, `@bullet-train/fields`). This skill only links the Ruby gems; changes to those packages' JavaScript in the clone are not picked up. For that, use the starter's `bin/hack --watch-js <package>` and `bin/hack --clean-js <package>`, which link the packages with yalc. `<package>` is the gem name (for example `bullet_train-fields`), not the npm name. `--clean-js` runs `yarn add` for the npm package, so it changes `package.json` and `yarn.lock`; review and discard those changes if you do not want them.

## Unlink

To go back to the released gems:

1. In the Gemfile, remove the `, path: "local/bullet_train-core/<folder>"` option from each `bullet_train*` gem line that has it. Leave `BULLET_TRAIN_VERSION` and any other options or comments in place. (`bin/hack --reset` also does this, but it strips every option after the first comma, so use it only if the lines have nothing else on them.)
2. Run `bundle install` so the lockfile returns to the released sources, then restart `bin/dev` and the Rails server.
3. Leave `./local/bullet_train-core` in place. Only delete it if the user asks. Leave the `/local` entry in `.gitignore` as well.

## Important notes

- Folder names in `bullet_train-core` equal the gem names, hyphens included. Do not translate hyphens to underscores or the reverse.
- Never commit the linked Gemfile or its `Gemfile.lock` changes.
- Do not modify files in the clone unless the user is deliberately working on core; if they are, remind them that it is a detached HEAD at a tag and they need a branch before committing.

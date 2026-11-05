# Bullet Train Claude Code Plugin

Claude Code plugin with development tools for [Bullet Train](https://bullettrain.co) applications.

## Installation

```bash
# Add the marketplace
/plugin marketplace add bullet-train-co/bullet_train-claude_code

# Install the plugin
/plugin install bullet-train@bullet-train
```

## Available Skills

### `/bullet-train:link-core`

Clone `bullet_train-core` and link its gems for local development.

This skill:
1. Reads `BULLET_TRAIN_VERSION` from your `Gemfile`
2. Clones `bullet_train-core` to `./local/bullet_train-core` (GitHub CLI, then HTTPS, then SSH)
3. Checks out the tag matching your app's version
4. Adds a `path:` option to each `bullet_train*` gem line in the `Gemfile`
5. Runs `bundle install` to link everything

It can also unlink: it removes the `path:` options and restores the released gems.

Useful when you need to debug or contribute to Bullet Train core gems. For the interactive workflow, the starter's `bin/hack` is also available; the skill explains when to pick which.

### `/bullet-train:bullet-train-versions`

Background reference on starter toolchain versions (Ruby, Node, Rails, Devise, Tailwind), known-bad releases, and where the core gems live. Useful for upgrades and dependency bumps. It is a snapshot, so check your app's own version files before relying on it.

### `/bullet-train:bullet-train-conventions`

Conventions for working inside a Bullet Train app: Super Scaffolding, finding and overriding gem code (`bin/resolve`), themes, the starter's scripts, API docs, and running tests.

## License

MIT

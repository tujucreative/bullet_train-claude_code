---
name: bullet-train-conventions
description: Conventions for working inside a Bullet Train (Rails) app. Use when adding models or fields (Super Scaffolding, rails generate super_scaffold, field types, generator flags), when scoping models to Team or editing permissions (roles.yml, ability.rb, magic comments), when something comes from a gem and must be found, overridden or ejected (bin/resolve, concerns), when changing a theme (bin/theme, theme eject/install), when generating OpenAPI docs, or when running the starter's tests.
---

# Bullet Train Conventions

Snapshot of bullet_train / bullet_train-core `v1.45.1`. Most of the framework lives in gems, not in the app; the app is a thin starter that includes gem concerns. Work with that, not around it.

## Rules that prevent the usual mistakes

1. **Scaffold, do not hand-write CRUD.** Run `rails generate super_scaffold` (or `super_scaffold:field` for new attributes), then `rake db:migrate`. Hand-written models skip the API controller, serializer, locale file, `roles.yml` permissions, breadcrumbs, routes and tests the scaffolder produces.
2. **Every scaffolded model is owned through `Team`.** The second argument is the chain of parents back to `Team`, for example `Goal Project,Team`. Controllers use `account_load_and_authorize_resource` and the ability comes from `config/models/roles.yml` plus `permit user, through: :memberships, parent: :team` in `app/models/ability.rb`; do not write ad-hoc `can` rules for scaffolded models.
3. **Assign work to `Membership`, not `User`** (`lead_id:super_select{class_name=Membership}`), and define the `valid_*` method the scaffolder asks for (for example `team.memberships.current_and_invited`). It feeds both the select and the security check.
4. **Never edit an installed gem.** Find the file with `bin/resolve`, then either redefine the method in the local stub (`class User < ApplicationRecord; include Users::Base; ...`) or copy it into the app with `--eject`. Ejected files stop receiving upstream updates, so prefer the stub. To change core itself, link a clone (see the `link-core` skill) and work there.
5. **Keep the `# 🚅 ...` magic comments** in models, controllers, views, locales and `ability.rb`. The scaffolder inserts at them; without them later `super_scaffold:field` runs print "We weren't able to find a super scaffolding hook".
6. **Commit before scaffolding.** Generated runs touch many files; the USAGE text recommends `git checkout .` and `git clean -d -f` to undo.
7. **`bin/super-scaffold` is deprecated.** It only re-execs `rails generate super_scaffold…`; use the generator directly.
8. **All `bullet_train*` gems share one version** (`BULLET_TRAIN_VERSION` in the Gemfile). Upgrade one version at a time by merging the starter tag (`git merge v1.4.1` style), then run tests. Version facts and bad releases: see the `bullet-train-versions` skill. Linking core locally: see `link-core`.

## Super Scaffolding

Generators (`rails generate super_scaffold` with no arguments prints help):

| Command | Does |
|---|---|
| `rails generate super_scaffold Model Parent,...,Team attr:type ...` | CRUD, API, locales, permissions, routes |
| `rails generate super_scaffold:field Model attr:type ...` | adds attributes to an existing scaffold |
| `rails generate super_scaffold:join_model Join::Model a_id{class_name=A} b_id{class_name=B}` | prepares a has-many-through join for a `_ids:super_select` field |
| `rails generate super_scaffold:incoming_webhooks`, `super_scaffold:oauth_provider` | webhooks / OAuth; run with `--help` |
| `rails generate super_scaffold:audit_logs`, `super_scaffold:conversations` | need the Audit Logs / Conversations gems |
| `rails generate super_scaffold:action_models:<type>` with `targets_many`, `targets_one`, `targets_one_parent`, `performs_export`, `performs_import` | need the Action Models gem |

Field types (`super_scaffold:field` USAGE, name: column): `address_field`, `boolean`, `buttons`, `cloudinary_image`, `color_picker`, `date_and_time_field`, `date_field`, `email_field`, `emoji_field`, `file_field` (attachment), `image` (attachment), `options`, `password_field`, `phone_field`, `super_select`, `text_area`, `text_field`, `number_field` (integer), `trix_editor`, `code_editor`. Attribute options go in braces: `{multiple}` (stored as jsonb; `attachments` for `file_field` and `image`), `{readonly}`, `{class_name=Membership}`, `{language=ruby}`, and quoted when they contain commas: `lead_id:super_select{"class_name=Membership,source=team.memberships.current_and_invited"}`.

Flags are parsed from raw `--name` / `--name=value` arguments at the end of the command (`lib/scaffolding/script.rb`): `--skip-migration-generation` (you ran `rails g model` yourself; do not migrate before scaffolding, the scaffolder may edit the migration), `--sortable`, `--namespace=customers`, `--navbar="ti-world"` (top-level models only; the generator's `--help` lists this as `--sidebar`, but the code reads `navbar`), `--only-index`, `--skip-views`, `--skip-form`, `--skip-locales`, `--skip-api`, `--skip-model`, `--skip-controller`, `--skip-routes`, `--skip-parent`; `super_scaffold:field` also has `--skip-show` and `--skip-table`. Hide the `Scaffolding::*` demo models from users with `HIDE_THINGS: true` in `config/application.yml`.

## Finding and overriding framework code

- `bin/resolve Account::Users::ControllerBase --open` prints the gem path of a class/module; `bin/resolve shared/box` resolves a partial; `bin/resolve en.account.onboarding.user_details.edit.header --eject --open` resolves a translation key. `--eject` copies the file into the app with an "Ejected from" header and refuses if the target exists unless `--force`. `--interactive` accepts a `<!-- BEGIN ... -->` comment pasted from the rendered HTML (view annotation is on in development only, `config/environments/development.rb`).
- `rake bullet_train:eject[locales]` copies every core locale file into `config/locales`. (`docs/indirection.md` names this `bullet_train:themes[eject]`; that task does not exist.)
- `?show_locales=true` or `?log_locales=true` on any URL shows the translation keys in use.

## Themes

- `bin/theme <file-type> <theme>` prints the path of a theme file, looking in the app first then the `bullet_train-themes-<theme>` gem. File types: `tailwind-config`, `tailwind-mailer-config`, `tailwind-stylesheet`, `stylesheets-dir`, `javascript`, `postcss-import-config`. The `package.json` CSS scripts call it (`bundle exec bin/theme tailwind-stylesheet light`); an ejected theme that lacks `postcss-import-config.js` needs `bin/resolve postcss-import-config.js --eject`.
- Forking the light theme: `rake bullet_train:themes:light:eject[custom_theme_name_here]`, then `rake bullet_train:themes:light:install` (rewrites `app/helpers/application_helper.rb`, `Procfile.dev`, `package.json` to the new theme name). Also `bullet_train:themes:light:eject_css`, `:clean[theme]` (lists unchanged ejected partials), `:release[theme_name]`.

## Starter scripts

- `bin/configure`: one-time, after cloning the starter (renames `origin` to `bullet-train`, renames the app, optional deploy buttons, commits). `bin/setup` installs dependencies and prepares the database; `bin/dev` runs `bin/link` then `Procfile.dev` via overmind or foreman.
- `bin/link` (`rake bt:link`) symlinks gems into `tmp/gems` for Tailwind scanning; `bin/hack` clones and links core (`bin/hack --help`; covered by the `link-core` skill).
- `bin/ci` runs `config/ci.rb`: `bin/setup --skip-server`, `bin/rubocop`, `bin/importmap audit`, `bin/rails test`, `env RAILS_ENV=test bin/rails db:seed:replant`; system tests are commented out there. It is the Rails default template: the starter ships no `bin/importmap`, so expect that step to fail until the app's own CI config is adjusted.

## API docs

- `rake bullet_train:api:generate_static_openapi` writes `public/api/<version>/openapi.yaml` for every version in `BulletTrain::Api.all_versions`. `rake bullet_train:api:export_openapi_schema` writes a timestamped copy under `tmp/openapi/`. `rake bullet_train:api:bump_version` copies `v1` controllers, views, tests and routes to the next version. `rake bullet_train:api:create_translations` adds `api_title`/`api_description` keys (anchored to each field's `heading`) to every field in the locale files, backing them up to `tmp/locales_backup` first.

## Tests

- `rails test` and `rails test:system` (docs/testing.md); the starter uses Minitest with FactoryBot (`test/factories`) and loads `db/seeds` in `test_helper.rb`. `MAGIC_TEST=1 rails test` shows the browser for Capybara tests.
- Super Scaffolding system tests need generated code first: `./test/system/super_scaffolding/<name>/setup.rb`, then `rails test test/system/super_scaffolding/<name>/`, then `./test/system/super_scaffolding/<name>/teardown.rb`, from a clean workspace. (`docs/testing.md` still names `test/bin/setup-super-scaffolding-system-test`, which the starter no longer ships.)

## Do not guess

If a generator flag, field option or rake task is not listed above, do not infer it. Run `rails generate super_scaffold --help` (or `super_scaffold:<type> --help`), `rake -T bullet_train`, `bin/resolve` with no arguments, or read `docs/` in the `bullet_train` gem.

## Sources (usage lines quoted verbatim)

- `bullet_train/bin/super-scaffold`: "`bin/super-scaffold` is deprecated. Please use `rails generate super_scaffold` instead."
- `bullet_train-core/bullet_train-super_scaffolding/lib/generators/super_scaffold/USAGE`: "rails g super_scaffold Site Team name:text_field url:text_area"; "rails g super_scaffold Section Page,Site,Team title:text_field body:text_area"; "rails g super_scaffold Page Site,Team name:text_field path:text_area --sortable"
- `.../super_scaffold/field/USAGE`: "bin/rails generate super_scaffold:field Project description:trix_editor" and the "Field Partial Types" list
- `.../super_scaffold/join_model/USAGE`: "rails g super_scaffold:field Project tag_ids:super_select{class_name=Projects::Tag}". The `join_model` and `incoming_webhooks` USAGE files show a stale `rails g bullet_train:...` prefix; the generators' `namespace` lines are `super_scaffold:join_model` and `super_scaffold:incoming_webhooks`
- `.../super_scaffold/audit_logs/USAGE`, `conversations/USAGE`, `action_models/*/USAGE`; the `*_generator.rb` files guard on `defined?(BulletTrain::AuditLogs)`, `BulletTrain::Conversations`, `BulletTrain::ActionModels`
- `.../super_scaffold/super_scaffold_generator.rb` and `field/field_generator.rb` (class options); `lib/scaffolding/script.rb` lines 34-48 (flag parsing); `lib/scaffolding/transformer.rb` line 1582 (`cli_options["navbar"]`)
- `bullet_train-core/bullet_train/docs/super-scaffolding.md`, `super-scaffolding/options.md`, `super-scaffolding/targets.md`, `field-partials.md`, `namespacing.md`, `modeling.md`, `indirection.md`, `overriding.md`, `testing.md`, `upgrades.md`
- `bullet_train-core/bullet_train/lib/tasks/bullet_train_tasks.rake`: "Usage: `bin/resolve [path, partial, or URL] (--eject) (--open)` OR `bin/resolve --interactive`"; task `bullet_train:eject[type]`; `lib/bullet_train/resolver.rb` (`--force`)
- `bullet_train/bin/resolve`, `bin/hack` (exec `rake bullet_train:resolve[...]` / `bullet_train:hack[...]`), `bin/link` ("bundle exec bin/rake bt:link"), `bin/dev`, `bin/setup`, `bin/configure`, `bin/ci`, `config/ci.rb`, `Procfile.dev`
- `bullet_train/bin/theme`: "i.e. - `bin/theme tailwind-config light`"; error text "bin/resolve postcss-import-config.js --eject"
- `bullet_train-core/bullet_train-themes-light/lib/tasks/bullet_train/themes/light_tasks.rake`; `bullet_train-themes/lib/tasks/application.rb` ("rake bullet_train:themes:light:eject[custom_theme_name_here]"); `bullet_train-themes/lib/tasks/bullet_train/themes_tasks.rake` (`bt:link`)
- `bullet_train-core/bullet_train-api/lib/tasks/bullet_train/api_tasks.rake`: tasks `bump_version`, `export_openapi_schema`, `generate_static_openapi`, `create_translations`
- `bullet_train/test/test_helper.rb`, `test/system/super_scaffolding/README.md`, `.github/workflows/_run_super_scaffolding_tests.yml`
- `bullet_train/app/models/ability.rb`, `config/models/roles.yml`, `app/models/user.rb`; `bullet_train-super_scaffolding/app/controllers/account/scaffolding/completely_concrete/tangible_things_controller.rb` (`account_load_and_authorize_resource`)

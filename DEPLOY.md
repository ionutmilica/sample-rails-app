# Deploy settings

Values to fill into the deployment form (Puma behind Nginx, Ruby runtime).

| Field             | Value                                                         |
| ----------------- | ------------------------------------------------------------- |
| Application port  | `3000`                                                        |
| Build command     | `SECRET_KEY_BASE_DUMMY=1 bundle exec rails assets:precompile` |
| Install command   | `bundle install`                                              |
| Publish directory | `.`                                                           |
| Runtime           | `ruby`                                                        |
| Ruby version      | `3.3`                                                         |
| Start command     | `bundle exec rails db:prepare && bundle exec puma -C config/puma.rb` |

## Why these values

- **Port 3000** — `config/puma.rb` binds to `ENV.fetch("PORT", 3000)`, so Nginx can proxy to Puma on 3000.
- **Build** — Propshaft precompiles assets into `public/assets`. `SECRET_KEY_BASE_DUMMY=1`
  lets the production environment boot for the asset task without needing real secrets.
- **Install** — standard Bundler install. `.ruby-version` pins Ruby `3.3`.
- **Publish directory `.`** — the whole app is the release artifact (no separate static output).
- **Start** — `db:prepare` creates/migrates the primary, cache, and queue SQLite databases on
  first boot, then Puma serves with the production config.

## Required environment variables

Set these in the platform's environment settings:

| Var                | Value                                    | Notes                                  |
| ------------------ | ---------------------------------------- | -------------------------------------- |
| `RAILS_ENV`        | `production`                             | Usually set by the platform already.   |
| `RAILS_MASTER_KEY` | contents of `config/master.key`          | Decrypts `config/credentials.yml.enc`. |

`config/master.key` is git-ignored — copy its contents into `RAILS_MASTER_KEY` on the platform.

## Notes

- `force_ssl` / `assume_ssl` are left off, so there is no redirect loop when Nginx terminates TLS.
- `config.hosts` is unset in production, so the proxied `Host` header is accepted.
- SQLite is used for app data, Solid Cache, and Solid Queue — fine for a single-node sample.
  Note the filesystem may be ephemeral on some platforms; use a persistent disk for real data.

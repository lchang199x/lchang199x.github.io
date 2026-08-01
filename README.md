# Chang's Homepage

> Blog of my engineering practices, stories and latest interests.

A personal blog built with [Jekyll](https://jekyllrb.com/) on the
[Hux Blog](https://github.com/huxpro/huxpro.github.io) theme, hosted on
GitHub Pages at **changliu.cc**.

## Local development

```bash
# 1. Install Ruby gems
bundle install

# 2. (Optional) provide Gitalk credentials for local comment preview
#    _config.local.yml is git-ignored and overrides _config.yml
cp /dev/null _config.local.yml
cat >> _config.local.yml <<YML
gitalk:
  clientID: "YOUR_OAUTH_CLIENT_ID"
  clientSecret: "YOUR_OAUTH_CLIENT_SECRET"
YML

# 3. Serve the site
bundle exec jekyll serve --config _config.yml,_config.local.yml
#    or simply:  npm run serve
```

The site is then available at <http://127.0.0.1:4000>.

## Security notes

Gitalk uses an OAuth *client secret*. **Never commit the real secret.**

- `_config.yml` keeps `clientID` / `clientSecret` blank.
- Real values live in the git-ignored `_config.local.yml` for local preview.
- For production, the GitHub Actions workflow (`.github/workflows/pages.yml`)
  injects the credentials from encrypted repository secrets
  (`GITALK_CLIENT_ID`, `GITALK_CLIENT_SECRET`) at build time.

> ⚠️ **Rotate the previously leaked secret.** An older revision exposed the
> Gitalk `clientSecret` in `_config.yml`. Go to *GitHub → Settings → Developer
> settings → OAuth Apps*, regenerate the client secret, and store the new value
> as a repository Action secret.

### Enabling Actions-based deployment

1. Add repository secrets `GITALK_CLIENT_ID` and `GITALK_CLIENT_SECRET`
   (*Settings → Secrets and variables → Actions*).
2. Set *Settings → Pages → Build and deployment → Source* to
   **GitHub Actions**.
3. Push to `main` — the workflow builds and deploys automatically.

## Maintenance

- **Dependabot** (`.github/dependabot.yml`) opens weekly PRs to bump Ruby gems,
  npm devDependencies, and GitHub Actions.
- The **CI workflow** builds the site on every pull request to catch Liquid /
  build errors before merging.

## Theme credits

Theme originates from [huxpro/huxpro.github.io](https://github.com/huxpro/huxpro.github.io)
(Clean Blog by Start Bootstrap), licensed under Apache 2.0.

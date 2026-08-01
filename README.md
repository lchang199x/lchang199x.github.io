# Chang's Homepage

> Blog of my engineering practices, stories and latest interests.

A personal blog built with [Jekyll](https://jekyllrb.com/) on the
[Hux Blog](https://github.com/huxpro/huxpro.github.io) theme, hosted on
GitHub Pages at **changliu.cc**.

## Local development

```bash
bundle install
bundle exec jekyll serve      # or: npm run serve
```

The site is then available at <http://127.0.0.1:4000>.

## Comments (Utterances)

Comments use [Utterances](https://utteranc.es) — a **secret-free** system backed
by GitHub Issues. Unlike the previous Gitalk setup, **no OAuth client secret is
exposed** anywhere (neither in the repo nor in the browser).

Configuration lives in `_config.yml` under the `utterances:` key. One-time setup:

1. Install the [utterances app](https://github.com/apps/utterances) and grant it
   access to `lchang199x/lchang199x.github.io`.
2. That's it — comments appear on the About page. Each page maps to a GitHub
   issue via `issue-term: pathname`.

> The old Gitalk OAuth client secret was removed entirely. You may delete the
> now-unused `GITALK_CLIENT_ID` / `GITALK_CLIENT_SECRET` repository secrets.

## Deployment

The site builds and deploys via GitHub Actions
(`.github/workflows/pages.yml`):

- On **pull request**: builds the site to catch Liquid / build errors.
- On **push to `main`**: builds and deploys to GitHub Pages.

Ensure *Settings → Pages → Build and deployment → Source* is set to
**GitHub Actions**.

## Maintenance

- **Dependabot** (`.github/dependabot.yml`) opens weekly PRs to bump Ruby gems,
  npm devDependencies, and GitHub Actions.
- The **CI workflow** builds the site on every pull request.

## Front-end stack

| Library | Version |
|---------|---------|
| Jekyll | 4.4.1 |
| jQuery | 3.7.1 |
| Bootstrap | 3.4.1 |
| Font Awesome | 4.7.0 (CDN + SRI) |
| FastClick | 1.0.6 (CDN + SRI) |

Third-party CDN resources are pinned to exact versions and protected with
Subresource Integrity (SRI) hashes.

## Theme credits

Theme originates from [huxpro/huxpro.github.io](https://github.com/huxpro/huxpro.github.io)
(Clean Blog by Start Bootstrap), licensed under Apache 2.0.

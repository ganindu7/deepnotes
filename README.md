# deepnotes

website url: https://ganindu7.github.io/deepnotes/

The site is built with [Jekyll](https://jekyllrb.com/docs/) and the
[Just the Docs](https://just-the-docs.com/) theme. Pushing to the `gh-pages`
branch runs `.github/workflows/pages.yml`, which builds the site and deploys
it to GitHub Pages.

## Important 
For this to work locally internet connectivity is needed

## Running locally

The Ruby version is pinned in `.ruby-version` (use rbenv).

1. Install rbenv (from git repo).
2. Install the pinned Ruby: `rbenv install` (reads `.ruby-version`).
3. Install the gems: `bundle install`
4. Serve the site with live reload:

```
bundle exec jekyll serve --livereload
```

then open http://127.0.0.1:4000/deepnotes/

With more custom options:

```
bundle exec jekyll serve --livereload --livereload_port 4002 --trace --port 4001
```

### Without a local Ruby (Docker)

```
docker run --rm -it -p 4000:4000 -v "$PWD":/site -v deepnotes_gems:/usr/local/bundle -w /site ruby:3.3 \
  bash -c 'bundle install && bundle exec jekyll serve --host 0.0.0.0'
```

then open http://127.0.0.1:4000/deepnotes/
